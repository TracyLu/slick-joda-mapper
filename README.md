# slick-joda-mapper

[![Build Status](https://travis-ci.org/tototoshi/slick-joda-mapper.png)](https://travis-ci.org/tototoshi/slick-joda-mapper)

Enables you to use joda-time with Slick.
You can persist `DateTime`, `Instant`, `LocalDateTime`, `LocalDate`, `LocalTime` with Slick.

# Usage

## For Slick 1.x

```scala
libraryDependencies += "com.github.tototoshi" %% "slick-joda-mapper" % "0.4.1"
```

```scala
import com.github.tototoshi.slick.JodaSupport._
```

## For Slick 2.x
Snapshot version is built with 2.0.0-RC1 now.
Please build from source if you are in trouble with the problem related to compatibility.

```scala
libraryDependencies ++= Seq(
    "com.typesafe.slick" %% "slick" % "2.0.0",
    "joda-time" % "joda-time" % "2.3", // Joda dependencies is now in `provided` scope
    "org.joda" % "joda-convert" % "1.5",
    "com.github.tototoshi" %% "slick-joda-mapper" % "1.0.0"
)
```

Import xJodaSupport(H2JodaSupport, PostgresJodaSupport, MySQLJodaSupport...) class suitable for the database you use.

```scala
import scala.slick.driver.H2Driver.simple._
import com.github.tototoshi.slick.H2JodaSupport._
```

# Example


```scala
scala> import com.github.tototoshi.slick.JodaSupport._
import com.github.tototoshi.slick.JodaSupport._

scala> import scala.slick.driver.PostgresDriver.simple._
import scala.slick.driver.PostgresDriver.simple._

scala> import org.joda.time.{ DateTime, LocalDate }
import org.joda.time.{DateTime, LocalDate}

scala>

scala> object JodaTest extends Table[(LocalDate, DateTime, Option[LocalDate], Option[DateTime])]("joda_test") {
     |   def localDate = column[LocalDate]("local_date")
     |   def dateTime = column[DateTime]("date_time")
     |   def optLocalDate = column[Option[LocalDate]]("opt_local_date")
     |   def optDateTime = column[Option[DateTime]]("opt_date_time")
     |   def * = localDate ~ dateTime ~ optLocalDate ~ optDateTime
     | }
defined module JodaTest

scala>

scala> val db = Database.forURL("jdbc:postgresql://localhost:5432/example", driver = "org.postgresql.Driver", user = "toshi")
db: scala.slick.session.Database = scala.slick.session.Database$$anon$2@62d37dc5

scala> db withSession { implicit session: Session =>
     |   JodaTest.ddl.create
     | }
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

scala> db withSession { implicit session: Session =>
     |   JodaTest.insert(new LocalDate(2012, 12, 4),
     |                   new DateTime(2012, 12, 4, 0, 0, 0, 0),
     |                   Some(new LocalDate(2012, 12, 5)),
     |                   None)
     | }
res1: Int = 1

scala> db withSession { implicit session: Session =>
     |   (for { j <- JodaTest } yield j).firstOption.foreach(println)
     | }
(2012-12-04,2012-12-04T00:00:00.000+09:00,Some(2012-12-05),None)
```


```
example=# \d joda_test
                 Table "public.joda_test"
     Column     |            Type             | Modifiers
----------------+-----------------------------+-----------
 local_date     | date                        | not null
 date_time      | timestamp without time zone | not null
 opt_local_date | date                        |
 opt_date_time  | timestamp without time zone |

example=# SELECT * FROM joda_test;
 local_date |      date_time      | opt_local_date | opt_date_time
------------+---------------------+----------------+---------------
 2012-12-04 | 2012-12-04 00:00:00 | 2012-12-05     |
(1 row)
```
