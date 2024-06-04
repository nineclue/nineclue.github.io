---
layout: post
title: Scala와 Postgresql에서 Enum 사용하기
date: 2024-06-03 15:59
category: programming
tags: scala
---

앞선 포스팅에서 class의 subclass를 postgresql에 enum type으로 저장하고 이를 다시 불러오는 예제를 작성해 보았습니다만 Scala에서 상관의 자료형을 db에 저장되는 Int로 가지고 있는 것이 마음에 걸립니다. 이를 Employee형으로 바꾸어 봅시다.

먼저 `ERef`형을 `Employee`로 바꾸어 주고 자료를 이에 맞게 바꾸어 줍니다.
```scala
object ScalaTypes:
    type ERef = Employee

object PgEnumTest extends IOApp:
    val es /*: Seq[Employee]*/ = 
        val b = Boss(0, "나사장", "000")
        val m1 = Manager(1, "너과장", "777", Option(b))
        val m2 = Manager(4, "또과장", "7777", Option(b))
        Seq(
            b,
            m1,
            Worker(2, "나만평사원", "123", Option(m1)),
            Worker(3, "한명더추가", "124", Option(m1)),
            m2,
            Worker(5, "과장많다", "133", Option(m2)),
            Worker(6, "고마뽑자", "134", Option(m2)),
            Worker(7, "그럴수가", "135", Option(m2))
        )
```

이래저래 찾아보았지만 postgres에서 scala 자료형을 만드는 방법을 없는것 같습니다. 그러면 Postgresql에 저장된 자료형을 읽어들여 scala형으로 바꾸어주는 함수를 만들어줘야 되겠습니다. 이전 버전에서는 바로 scala class로 변환했지만 이번에는 따로 `EmpRaw` 자료형을 만들어 이 형태로 읽은 다음 id에 맞추어 class를 생성하도록 만들어 줍니다. postgresql에서의 query는 이전과 같이 단순한 query가 아니라 ref가 null인 가장 상위에서 시작해서 그 상위 row의 eid 값을 ref로 가지는 row들을 다음에 읽어 상위에서 점차 하위로 읽어나가는 [recursive common table expression](https://www.postgresql.org/docs/16/queries-with.html)을 사용하도록 하겠습니다. 

```scala
object PgEnumTest extends IOApp:
    val recQuery = 
        sql"""WITH RECURSIVE emps(eid, emptype, name, emp_number, sup) AS (
                | SELECT e.eid, subtype, name, emp_number, ref
                | FROM employee e
                | WHERE ref IS NULL 
                | UNION ALL 
                | SELECT e2.eid, e2.subtype, e2.name, e2.emp_number, e2.ref
                | FROM employee e2
                | JOIN emps ee ON e2.ref = ee.eid
                | )
                | SELECT * from EMPS;
        """.stripMargin.query[EmpRaw]
            /*
            eid | emptype |    name    | sup
            -----+---------+------------+-----
            0 | 사장    | 나사장     |
            1 | 과장    | 너과장     |   0
            4 | 과장    | 또과장     |   0
            2 | 평사원  | 나만평사원 |   1
            3 | 평사원  | 한명더추가 |   1
            5 | 평사원  | 과장많다   |   4
            6 | 평사원  | 고마뽑자   |   4
            7 | 평사원  | 그럴수가   |   4
            */

    def retrieve2 = recQuery.to[List].transact(xa)

object ScalaTypes:
    type EmpRaw = (Int, EmployeeTypes, String, String, Option[Int])

    // Read[Employee]는 코멘트하거나 삭제
    given Write[Employee] = Write[EmpRaw].contramap: 
        _ match
            case Worker(eid, name, eno, oref) => (eid, EmployeeTypes.Worker, name, eno, oref.map(_.getEid))
            case Manager(eid, name, eno, oref) => (eid, EmployeeTypes.Manager, name, eno, oref.map(_.getEid))
            case Boss(eid, name, eno) => (eid, EmployeeTypes.Boss, name, eno, None)
```

> ##### TIP
> 
> [Postgresql의 문서](https://www.postgresql.org/docs/16/queries-with.html)에 따르면 recursive query의 순서는 vendor에 따라 다를 수 있으며 postgresql은 기본적으로 breadth-first search를 사용한다고 되어 있습니다. 
> 
> 내부 구현이 달라져 query 결과의 순서가 바뀐다면 이에 맞게 scala code도 수정해야 하겠습니다.
{: .block-tip}

이제 postgres에서는 EmpRaw형의 list로 얻어왔고 이를 scala object로 만들어 봅시다. 
```scala
object PgEnumTest extends IOApp:
    def emap(src: EmpRaw, ps: Seq[Employee]) = src match
        case (eid, et, name, eno, oref) => 
            val sup = oref.flatMap(id => ps.find(_.getEid == id))
            et match
                case EmployeeTypes.Worker =>
                    Worker(eid, name, eno, sup)
                case EmployeeTypes.Manager =>
                    Manager(eid, name, eno, sup)
                case EmployeeTypes.Boss =>
                    Boss(eid, name, eno)

    def convert(es: List[EmpRaw]) =
        val ehead = emap(es.head, Seq.empty)
        es.tail.scanLeft((ehead, Seq(ehead))) :
            case (((_, pes), e)) => 
                val supNum = e._5.getOrElse(-1)
                val nes = pes.dropWhile(_.getEid != supNum)
                val n = emap(e, nes)
                (n, nes :+ n)
        .map(_._1)


    def run(as: List[String]): IO[ExitCode] = 
        for 
            _   <-  IO.println(es)
            _   <-  createTablesTypes 
            _   <-  store
            des <-  retrieve2
            _   <-  IO.println(convert(des))
        yield (ExitCode.Success)
```
`emap`함수는 삭제한 `Read[Employee]`와 유사하게 자료값을 받아 Employee를 만들어 줍니다. 단, 상위 후보들 목록을 추가 인자로 받아 eid가 같은 Employee를 찾아 ref값으로 지정합니다. `convert` 함수는 scanLeft를 사용해서 각 열들을 `emap` 함수로 변환하며 필요없는 상위들은 버리고 추가하는 동작을 합니다. 실행시켜 보면 처음 println(es)에서는 우리가 코드에서 지정한 boss, manager1, manager1의 worker들, manager2, manager2의 worker들 순서로 출력되지만 마지막의 println(convert(des))에서는 boss, manager1, manager2, worker들 순서로 출력되는 것을 확인할 수 있습니다.

원하는 출력을 얻었으나 자료를 전부 `List[EmpRaw]`로 얻어와서 이를 다시 scala로 바꾸는 것은 비효율적입니다. 자료를 읽으면서 변환하는 방법을 없을까요? doobie는 stream 명령으로 자료를 fs2의 stream으로 가져올 수 있어 다음과 같은 방법을 사용할 수 있습니다. 
```scala
object PgEnumTest extends IOApp:
    def retrieve3 = 
        val es = recQuery.stream
        case object Dummy extends Employee(0, "", "", None)
        // es.head는 Stream이라 앞서와 다른 방법을 사용
        es.scan((Dummy: Employee, Seq.empty[Employee])) :
            case (((_, pes), e)) =>
                val supNum = e._5.getOrElse(-1)
                val nes = pes.dropWhile(_.getEid != supNum)
                val n = emap(e, nes)
                (n, nes :+ n)
        .tail.map(_._1).compile.toList.transact(xa)

    def run(as: List[String]): IO[ExitCode] = 
        for 
            _   <-  IO.println(es)
            _   <-  createTablesTypes 
            _   <-  store
            des <-  retrieve3
            _   <-  IO.println(des)
        yield (ExitCode.Success)
```