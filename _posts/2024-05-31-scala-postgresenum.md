---
layout: post
title: Scala와 Postgres에서 Enum 사용
date: 2024-05-31 14:52
category: programming
tags: scala
---

[doobie](https://tpolecat.github.io/doobie/index.html)를 사용하는 경우 Scala의 case class는 자동으로 변환해서 query나 insert에서 사용할 수 있습니다. 몇가지 서로 연관된 class를 db에 저장할 일이 생겼는데 상속받는 class를 저장하는 경우 어떻게 처리해야 할지 찾아보면서 간만에 포스팅을 합니다. 

doobie는 [postgres 확장](https://tpolecat.github.io/doobie/docs/15-Extensions-PostgreSQL.html)을 지원하며 이는 배열, enum, geometry 등 여러가지 기능을 추가로 사용할 수 있도록 해 줍니다. 먼저 enum을 사용해서 상속받는 class를 사용해 보겠습니다.

간단히 자료는 다음과 같은 테이블과 enum으로 생성하고 저장했습니다.
```scala
object PgEnumTest extends IOApp:
    val xa = Transactor.fromDriverManager[IO](
        driver = "org.postgresql.Driver",  // JDBC driver classname
        url = "jdbc:postgresql:db이름",     // Connect URL - Driver specific
        user = "",                         // Database user name
        password = "",                     // Database password
        logHandler = None                  // Don't setup logging for now. See Logging page for how to log events in detail
    )

    def createTablesTypes = 
        Seq(
            sql"DROP TABLE IF EXISTS employee".update.run,
            sql"DROP TYPE IF EXISTS emptype".update.run,
            sql"CREATE TYPE emptype AS ENUM ('평사원', '과장', '사장')".update.run,
            sql"""CREATE TABLE IF NOT EXISTS employee 
            | (eid SERIAL PRIMARY KEY,
            | subtype emptype NOT NULL,
            | name TEXT NOT NULL, 
            | emp_number TEXT NOT NULL,
            | ref INTEGER REFERENCES employee (eid))
            """.stripMargin.update.run,
        ).traverse(_.transact(xa))

    def run(as: List[String]): IO[ExitCode] = 
        for 
            _   <-  createTablesTypes 
        yield (ExitCode.Success)

object ScalaTypes:
    enum EmployeeTypes:
        case Worker, Manager, Boss

    type ERef = Int
    class Employee(eid: Int, name: String, empNumber: String, ref: Option[ERef]):
        def getEid = eid 
    case class Worker(eid: Int, name: String, empNumber: String, ref: Option[ERef]) extends Employee(eid, name, empNumber, ref)
    case class Manager(eid: Int, name: String, empNumber: String, ref: Option[ERef]) extends Employee(eid, name, empNumber, ref)
    case class Boss(eid: Int, name: String, empNumber: String) extends Employee(eid, name, empNumber, None)
```
class이름과 enum 값을 동일하게 사용한 것도, 상속이 구질구질한 것도 마음에 들지 않지만 딱히 깔끔한 방법을 아직 찾지 못했습니다.

doobie에서는 postgres enum을 3가지 방법으로 지원하며 scala 3의 경우 `pgEnum`을 써도 제대로 변환되지 않아 아래와 같이 문자열을 바로 변환하는 `pgEnumStringOpt` 방식을 따랐습니다. 
```scala
object ScalaTypes:
    enum EmployeeTypes:
        case Worker, Manager, Boss

    // 앞선 예의 다음에 추가 
    def toEmpTypes(s: String): Option[EmployeeTypes] = s match
        case "평사원" => Option(EmployeeTypes.Worker)
        case "과장" => Option(EmployeeTypes.Manager)
        case "사장" => Option(EmployeeTypes.Boss)
        case _     => None
    def fromEmpTypes(st: EmployeeTypes): String = st match
        case EmployeeTypes.Worker => "평사원"
        case EmployeeTypes.Manager => "과장"
        case EmployeeTypes.Boss => "사장"
    given Meta[EmployeeTypes] = pgEnumStringOpt("emptype", toEmpTypes, fromEmpTypes)
```
위와 같이 작성하고 나면 scala의 EmployeeTypes와 postgres의 emptype 사이의 변환을 doobie에서 자동으로 하게 됩니다. 적어놓고 보니 String 값과 text 필드로 변환해도 비슷했을 것 같은데 enum을 사용하는 경우 postgres에서 저장하는 text 값을 한번 더 확인하고 enum에 순서가 지정되는 장점이 있다 합니다.

이제 scala의 Employee와 이하 상속 class들과 postgresql의 변환을 아래와 같이 `Read`, `Write`를 사용하여 수동으로 지정합니다. 자세한 것은 [클래스의 custom mapping](https://tpolecat.github.io/doobie/docs/12-Custom-Mappings.html)을 참고하시면 되겠습니다.
```scala
object ScalaTypes:
    // 앞선 예의 다음에 추가 
    given Read[Employee] = Read[(Int, EmployeeTypes, String, String, Option[Int])].map:
        case (eid, et, name, eno, oref) => 
            et match
                case EmployeeTypes.Worker =>
                    Worker(eid, name, eno, oref)
                case EmployeeTypes.Manager =>
                    Manager(eid, name, eno, oref)
                case EmployeeTypes.Boss =>
                    Boss(eid, name, eno)

    given Write[Employee] = Write[(Int, EmployeeTypes, String, String, Option[Int])].contramap: 
        _ match
            case Worker(eid, name, eno, oref) => (eid, EmployeeTypes.Worker, name, eno, oref)
            case Manager(eid, name, eno, oref) => (eid, EmployeeTypes.Manager, name, eno, oref)
            case Boss(eid, name, eno) => (eid, EmployeeTypes.Boss, name, eno, None)
```
위의 `Read`는 database에서 변환된 EmplyoeeTypes enum 값에 따라 Emplyoee의 상속 클래스중 하나를 만들어 반환하고 아래의 `Write`는 반대로 상속 class를 받아 database에서 저장할 수 있는 tuple 형태로 지정합니다.

이제 변환 준비가 되었으니 `PgEnumTest` object 아래에 값들을 쓰고 읽는 함수를 추가해 봅시다.
```scala
object PgEnumTest extends IOApp:
    // run 함수 위에 입력하고 run 함수를 수정합니다.
    import ScalaTypes.*
    val es /*: Seq[Employee]*/ = Seq(
        Boss(0, "나사장", "000"),
        Manager(1, "너과장", "777", Option(0)),
        Worker(2, "나만평사원", "123", Option(1)),
        Worker(3, "한명더추가", "124", Option(1)),
        Manager(4, "또과장", "7777", Option(0)),
        Worker(5, "과장많다", "133", Option(4)),
        Worker(6, "고마뽑자", "134", Option(4)),
        Worker(7, "그럴수가", "135", Option(4))
    )

    def store =
        val inserts = "INSERT INTO employee VALUES (?, ?, ?, ?, ?)"
        Update[Employee](inserts).updateMany(es).transact(xa)

    def retrieve = 
        sql"SELECT * FROM employee".query[Employee].to[List].transact(xa)

    def run(as: List[String]): IO[ExitCode] = 
        for 
            _   <-  createTablesTypes 
            _   <-  store
            des <-  retrieve
            _   <-  IO.println(des)
        yield (ExitCode.Success)    
```
Emplyoee의 하위 class들을 성공적으로 database에 저장하고 또 읽어오는 코드를 작성했습니다. 하지만 Manager와 Worker class는 내부적으로 database에 저장되는 eid 값을 포함하고 있습니다. 직접 object를 사용하도록 할 수 있을까요? `Write`는 간단히 작성할 수 있지만 `Read`는 어떻게 작성해야 할까요? 연구중입니다. 조만간 update 할 수 있기를 바라면서.