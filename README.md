# Squants

**The Scala API for Quantities, Units of Measure and Dimensional Analysis**

Squants is a framework of data types and a domain specific language (DSL) for representing Quantities,
their Units of Measure, and their Dimensional relationships.
The API supports typesafe dimensional analysis, improved domain models and more.
All types are immutable and thread-safe.

[Website](http:/www.squants.com/)
|
[GitHub](https://github.com/typelevel/squants)
|
[Wiki](https://github.com/typelevel/squants/wiki)

[![Join the chat at https://gitter.im/garyKeorkunian/squants](https://badges.gitter.im/garyKeorkunian/squants.svg)](https://gitter.im/garyKeorkunian/squants?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

### Current Versions
Current Release: **1.1.0**
([API Docs](https://oss.sonatype.org/service/local/repositories/releases/archive/org/typelevel/squants_2.11/1.1.0/squants_2.11-1.1.0-javadoc.jar/!/index.html#squants.package))

Development Build: **1.2.0-SNAPSHOT**
([API Docs](https://oss.sonatype.org/service/local/repositories/snapshots/archive/org/typelevel/squants_2.11/1.2.0-SNAPSHOT/squants_2.11-1.2.0-SNAPSHOT-javadoc.jar/!/index.html#squants.package))

[Release History](https://github.com/typelevel/squants/wiki/Release-History)

[![Build Status](https://travis-ci.org/typelevel/squants.png?branch=master)](https://travis-ci.org/typelevel/squants)

Build services provided by [Travis CI](https://travis-ci.com/)

NOTE - This README reflects the feature set in the branch it can be found.
For more information on feature availability of a specific version see the Release History or the README for a that version

## Installation
Repository hosting for Squants is provided by [Sonatype](https://oss.sonatype.org/).
To use Squants in your SBT project add the following dependency to your build.

    "org.typelevel"  %% "squants"  % "1.1.0"
or

    "org.typelevel"  %% "squants"  % "1.2.0-SNAPSHOT"


To use Squants in your Maven project add the following dependency

```xml
<dependency>
    <groupId>org.typelevel</groupId>
    <artifactId>squants_2.11</artifactId>
    <version>1.1.0</version>
</dependency>
```

Beginning with Squants 0.4.x series, both Scala 2.10 and 2.11 builds are available.
Beginning with Squants 1.0.0 series, Scala 2.10, 2.11 and 2.12 builds are available.

To use Squants interactively in the Scala REPL, clone the git repo and run `sbt squantsJVM/console`

    git clone https://github.com/typelevel/squants
    cd squants
    sbt squantsJVM/console

## Type Safe Dimensional Analysis
*The Trouble with Doubles*

When building programs that perform dimensional analysis, developers are quick to declare
quantities using a basic numeric type, usually Double.  While this may be satisfactory in some situations,
it can often lead to semantic and other logic issues.

For example, when using a Double to describe quantities of Energy (kWh) and Power (kW), it is possible
to compile a program that adds these two values together.  This is not appropriate as kW and kWh
measure quantities of two different dimensions.  The unit kWh is used to measure an amount of Energy used
or produced.  The unit kW is used to measure Power/Load, the rate at which Energy is being used
or produced, that is, Power is the first time derivative of Energy.

*Power = Energy / Time*

Consider the following code

```scala
val loadKw = 1.2                    // Double: 1.2
val energyMwh = 24.2                // Double: 24.2        
val sumKw = loadKw + energyMwh      // Double: 25.4
```

which not only adds quantities of different dimensions (Power vs Energy),
it also fails to convert the scales implied in the val names (Mega vs Kilo).
Because this code compiles, detection of these errors is pushed further into the development cycle.

### Dimensional Type Safety

_Only quantities with the same dimensions may be compared, equated, added, or subtracted._

Squants helps prevent errors like these by type checking operations at compile time and
automatically applying scale and type conversions at run-time.  For example,

```scala
import squants.energy.{Kilowatts, Megawatts, Power}
val load1: Power = Kilowatts(12)        // returns Power(12, Kilowatts) or 12 kW
val load2: Power = Megawatts(0.023)     // Power: 0.023 MW
val sum = load1 + load2                 // Power: 35 kW - unit on left side is preserved
sum should be(Kilowatts(35))            
sum should be(Megawatts(0.035))         // comparisons automatically convert scale
```

works because Kilowatts and Megawatts are both units of Power.  Only the scale is
different and the library applies an appropriate conversion.  Also, notice that keeping track of
the scale within the value name is no longer needed.

```scala
scala> import squants.energy.{Energy, Power, Kilowatts, KilowattHours}
import squants.energy.{Energy, Power, Kilowatts, KilowattHours}

scala> val load: Power = Kilowatts(1.2)            // Power: 1.2 kW
load: squants.energy.Power = 1.2 kW

scala> val energy: Energy = KilowattHours(23.0)    // Energy: 23 kWH
energy: squants.energy.Energy = 23.0 kWh
```

```scala
scala> val sum = load + energy                     // Invalid operation - does not compile
<console>:15: error: type mismatch;
 found   : squants.energy.Energy
 required: squants.energy.Power
       val sum = load + energy                     // Invalid operation - does not compile
                        ^
```
The unsupported operation in this expression prevents the code from compiling,
catching the error made when using Double in the example above.

### Dimensionally Correct Type Conversions

_One may take quantities with different dimensions, and multiply or divide them._

Dimensionally correct type conversions are a key feature of Squants.
Conversions are implemented by defining relationships between Quantity types using the * and / operators.

The following code demonstrates creating ratio between two quantities of the same dimension,
resulting in a dimensionless value:

```scala
import squants.time.{Hours, Days}

val ratio = Days(1) / Hours(3)  // Double: 8.0
```

This code demonstrates use of the `Power.*` method that takes a `Time` and returns an `Energy`:

```scala
val load = Kilowatts(1.2)                   // Power: 1.2 kW
val time = Hours(2)                         // Time: 2 h
val energyUsed = load * time                // Energy: 2.4 kWh
```

This code demonstrates use of the `Energy./` method that takes a `Time` and returns a `Power`:

```scala
val aveLoad: Power = energyUsed / time      // Power: 1.2 kW
```

### Unit Conversions

Quantity values are based in the units used to create them.

```scala
import squants.energy.{Kilowatts, Megawatts}

val loadA: Power = Kilowatts(1200)  // Power: 1200.0 kW
val loadB: Power = Megawatts(1200)  // Power: 1200.0 MW
```

Since Squants properly equates values of a like dimension, regardless of the unit,
there is usually no reason to explicitly convert from one to the other.
This is especially true if the user code is primarily performing dimensional analysis.

However, there are times when you may need to set a Quantity value to a specific unit (eg, for proper JSON encoding).

When necessary, a quantity can be converted to another unit using the `in` method.

```scala
scala> import squants.energy.{Gigawatts, Kilowatts, Megawatts}
import squants.energy.{Gigawatts, Kilowatts, Megawatts}

scala> val loadA = Kilowatts(1200)    // Power: 1200.0 kW
loadA: squants.energy.Power = 1200.0 kW

scala> val loadB = loadA in Megawatts // Power: 1.2 MW
loadB: squants.energy.Power = 1.2 MW

scala> val loadC = loadA in Gigawatts // Power: 0.0012 GW
loadC: squants.energy.Power = 0.0012 GW
```

Sometimes you need to get the numeric value of the quantity in a specific unit
(eg, for submission to an external service that requires a numeric in a specified unit
or to perform analysis beyond Squant's domain)

When necessary, the value can be extracted in the desired unit with the `to` method.

```scala
import scala.language.postfixOps

val load: Power = Kilowatts(1200)
val kw: Double = load to Kilowatts // Double: 1200.0
val mw: Double = load to Megawatts // Double: 1.2
val gw: Double = load to Gigawatts // Double: 0.0012
```

Most types include methods with convenient aliases for the `to` methods.

```scala
val kw: Double = load toKilowatts // Double: 1200.0
val mw: Double = load toMegawatts // Double: 1.20
val gw: Double = load toGigawatts // Double: 0.0012
```

NOTE - It is important to use the `to` method for extracting the numeric value,
as this ensures you will be getting the numeric value for the desired unit.
`Quantity.value` should not be accessed directly.
To prevent improper usage, direct access to the `Quantity.value` field may be deprecated in a future version.

Creating strings formatted in the desired unit:

```scala
scala> val kw: String = load toString Kilowatts // String: “1200.0 kW”
kw: String = 1200.0 kW

scala> val mw: String = load toString Megawatts // String: “1.2 MW”
mw: String = 1.2 MW

scala> val gw: String = load toString Gigawatts // String: “0.0012 GW”
gw: String = 0.0012 GW
```

Creating Tuple2(Double, String) that includes a numeric value and unit symbol:

```scala
scala> val load: Power = Kilowatts(1200)
load: squants.energy.Power = 1200.0 kW

scala> val kw = load toTuple               // Tuple2: (1200, "kW")
kw: (Double, String) = (1200.0,kW)

scala> val mw = load toTuple Megawatts     // Tuple2: (1.2, "MW)
mw: (Double, String) = (1.2,MW)

scala> val gw = load toTuple Gigawatts     // Tuple2: (0.0012, "GW")
gw: (Double, String) = (0.0012,GW)
```

This can be useful for passing properly scaled quantities to other processes
that do not use Squants, or require use of more basic types (Double, String)

Simple console based conversions (using DSL described below)

```scala
scala> import squants.mass.MassConversions._
import squants.mass.MassConversions._

scala> import squants.mass.{Kilograms, Pounds}
import squants.mass.{Kilograms, Pounds}

scala> import squants.thermal.TemperatureConversions._
import squants.thermal.TemperatureConversions._

scala> import squants.thermal.Fahrenheit
import squants.thermal.Fahrenheit

scala> 1.kilograms to Pounds       // Double: 2.2046226218487757
res3: Double = 2.2046226218487757

scala> kilogram / pound            // Double: 2.2046226218487757
res4: Double = 2.2046226218487757

scala> 2.1.pounds to Kilograms     // Double: 0.952543977
res5: Double = 0.952543977

scala> 2.1.pounds / kilogram       // Double: 0.952543977
res6: Double = 0.9525439770000002

scala> 100.C to Fahrenheit         // Double: 212.0
res7: Double = 212.0
```

### Mapping over Quantity values
Apply a `Double => Double` operation to the underlying value of a quantity, while preserving its type and unit.

```scala
scala> val load = Kilowatts(2.0)                   // 2.0 kW
load: squants.energy.Power = 2.0 kW

scala> val newLoad = load.map(v => v * 2 + 10)     // Power: 14.0 kW
newLoad: squants.energy.Power = 14.0 kW
```

The q.map(f) method effectively expands to q.unit(f(q.to(q.unit))

### Approximations
Create an implicit Quantity value to be used as a tolerance in approximations.
Then use the `approx` method (or `=~`, `~=`, `≈` operators) like you would use the `equals` method (`==` operator).

```scala
scala> import squants.energy.Watts
import squants.energy.Watts

scala> implicit val tolerance = Watts(.1)      // implicit Power: 0.1 W
tolerance: squants.energy.Power = 0.1 W

scala> val load = Kilowatts(2.0)               // Power: 2.0 kW
load: squants.energy.Power = 2.0 kW

scala> val reading = Kilowatts(1.9999)         // Power: 1.9999 kW
reading: squants.energy.Power = 1.9999 kW

scala>  // uses implicit tolerance
     | load =~ reading 
res9: Boolean = true

scala> load ≈ reading 
res10: Boolean = true

scala> load approx reading
res11: Boolean = true
```

The `=~` and `≈` are the preferred operators as they have the correct precedence for equality operations.
The `~=` is provided for those who wish to use a more natural looking approx operator using standard characters.
However, because of its lower precedence, user code may require parenthesis around these comparisons.

### Vectors

All `Quantity` types in Squants represent the scalar value of a quantity.
That is, there is no direction information encoded in any of the Quantity types.
This is true even for Quantities which are normally vector quantities (ie. Velocity, Acceleration, etc).

Vector quantities in Squants are implemented as case classes that takes a variable parameter list of like quantities
representing a set of point coordinates in Cartesian space.  
The SVector object is a factory for creating DoubleVectors and QuantityVectors.
The dimensionality of the vector is determined by the number of arguments.
Most basic vector operations are currently supported (addition, subtraction, scaling, cross and dot products)

```scala
scala> import squants.{QuantityVector, SVector}
import squants.{QuantityVector, SVector}

scala> import squants.space.{Kilometers, Length}
import squants.space.{Kilometers, Length}

scala> import squants.space.LengthConversions._
import squants.space.LengthConversions._

scala> val vector: QuantityVector[Length] = SVector(Kilometers(1.2), Kilometers(4.3), Kilometers(2.3))
vector: squants.QuantityVector[squants.space.Length] = QuantityVector(WrappedArray(1.2 km, 4.3 km, 2.3 km))

scala> val magnitude: Length = vector.magnitude        // returns the scalar value of the vector
magnitude: squants.space.Length = 5.021951811795888 km

scala> val normalized = vector.normalize(Kilometers)   // returns a corresponding vector scaled to 1 of the given unit
normalized: vector.SVectorType = QuantityVector(ArrayBuffer(0.2389509188800581 km, 0.8562407926535415 km, 0.45798926118677796 km))

scala> val vector2: QuantityVector[Length] = SVector(Kilometers(1.2), Kilometers(4.3), Kilometers(2.3))
vector2: squants.QuantityVector[squants.space.Length] = QuantityVector(WrappedArray(1.2 km, 4.3 km, 2.3 km))

scala> val vectorSum = vector + vector2        // returns the sum of two vectors
vectorSum: vector.SVectorType = QuantityVector(ArrayBuffer(2.4 km, 8.6 km, 4.6 km))

scala> val vectorDiff = vector - vector2       // return the difference of two vectors
vectorDiff: vector.SVectorType = QuantityVector(ArrayBuffer(0.0 km, 0.0 km, 0.0 km))

scala> val vectorScaled = vector * 5           // returns vector scaled 5 times
vectorScaled: vector.SVectorType = QuantityVector(ArrayBuffer(6.0 km, 21.5 km, 11.5 km))

scala> val vectorReduced = vector / 5          // returns vector reduced 5 time
vectorReduced: vector.SVectorType = QuantityVector(ArrayBuffer(0.24 km, 0.86 km, 0.45999999999999996 km))

scala> val vectorDouble = vector / 5.meters    // returns vector reduced and converted to DoubleVector
vectorDouble: squants.DoubleVector = DoubleVector(ArrayBuffer(240.0, 860.0, 459.99999999999994))

scala> val dotProduct = vector * vectorDouble  // returns the Dot Product of vector and vectorDouble
dotProduct: squants.space.Length = 5044.0 km

scala> val crossProduct = vector crossProduct vectorDouble  // currently only supported for 3-dimensional vectors
crossProduct: vector.SVectorType = QuantityVector(WrappedArray(0.0 km, 1.1368683772161603E-13 km, 0.0 km))
```

Simple non-quantity (Double based) vectors are also supported.

```scala
import squants.DoubleVector

val vector: DoubleVector = SVector(1.2, 4.3, 2.3, 5.4)   // a Four-dimensional vector
```

#### Dimensional conversions within Vector operations.
NOTE - This feature is currently under development and the final implementation being evaluated.
The following type of operation is the goal.

```scala
import squants.time.Seconds

val vectorLength = QuantityVector(Kilometers(1.2), Kilometers(4.3), Kilometers(2.3))
val vectorArea = vectorLength * Kilometers(2)   // QuantityVector(2.4 km², 8.6 km², 4.6 km²)
val vectorVelocity = vectorLength / Seconds(1)  // QuantityVector(1200.0 m/s, 4300.0 m/s, 2300.0 m/s)

val vectorDouble = DoubleVector(1.2, 4.3, 2.3)
val vectorLength = vectorDouble.to(Kilometers)  // QuantityVector(1.2 km, 4.3 km, 2.3 km)
```

Currently dimensional conversions are supported by using the slightly verbose, but flexible map method.

```scala
import squants.motion.Velocity
import squants.space.{Area, Length}

val vectorLength = QuantityVector(Kilometers(1.2), Kilometers(4.3), Kilometers(2.3))
val vectorArea = vectorLength.map[Area](_ * Kilometers(2))      // QuantityVector(2.4 km², 8.6 km², 4.6 km²)
val vectorVelocity = vectorLength.map[Velocity](_ / Seconds(1)) // QuantityVector(1200.0 m/s, 4300.0 m/s, 2300.0 m/s)

val vectorDouble = DoubleVector(1.2, 4.3, 2.3)
val vectorLength = vectorDouble.map[Length](Kilometers(_))      // QuantityVector(1.2 km, 4.3 km, 2.3 km)
```

Convert QuantityVectors to specific units using the `to` or `in` method - much like Quantities.

```scala
import squants.space.Meters

val vectorLength = QuantityVector(Kilometers(1.2), Kilometers(4.3), Kilometers(2.3))
val vectorMetersNum = vectorLength.to(Meters)   // DoubleVector(1200.0, 4300.0, 2300.0)
val vectorMeters = vectorLength.in(Meters)      // QuantityVector(1200.0 m, 4300.0 m, 2300.0 m)
```

## Market Package
Market Types are similar but not quite the same as other quantities in the library.
The primary type, Money, is a Dimensional Quantity, and its Units of Measure are Currencies.
However, because the conversion multipliers between currency units can not be predefined,
many of the behaviors have been overridden and augmented to realize correct behavior.

### Money
A Quantity of purchasing power measured in Currency units.
Like other quantities, the Unit of Measures are used to create Money values.

```scala
import squants.market.{BTC, JPY, USD, XAU}

val tenBucks = USD(10)      // Money: 10 USD
val someYen = JPY(1200)     // Money: 1200 JPY
val goldStash = XAU(50)     // Money: 50 XAU
val digitalCache = BTC(50)  // Money: 50 BTC
```

### Price
A Ratio between Money and another Quantity.
A Price value is typed on a Quantity and can be denominated in any defined Currency.

*Price = Money / Quantity*

```scala
scala> import squants.{Dozen, Each}
import squants.{Dozen, Each}

scala> import squants.energy.MegawattHours
import squants.energy.MegawattHours

scala> import squants.space.UsGallons
import squants.space.UsGallons

scala> val threeForADollar = USD(1) / Each(3)              // Price[Dimensionless]: 1 USD / 3 ea
threeForADollar: squants.market.Price[squants.Dimensionless] = 1.00 USD/3.0 ea

scala> val energyPrice = USD(102.20) / MegawattHours(1)    // Price[Energy]: 102.20 USD / megawattHour
energyPrice: squants.market.Price[squants.energy.Energy] = 102.20 USD/1.0 MWh

scala> val milkPrice = USD(4) / UsGallons(1)               // Price[Volume]: 4 USD / gallon
milkPrice: squants.market.Price[squants.space.Volume] = 4.00 USD/1.0 gal

scala> val costForABunch = threeForADollar * Dozen(10) // Money: 40 USD
costForABunch: squants.market.Money = 40.00 USD

scala> val energyCost = energyPrice * MegawattHours(4) // Money: 408.80 USD
energyCost: squants.market.Money = 408.80 USD

scala> val milkQuota = USD(20) / milkPrice             // Volume: 5 gal
milkQuota: squants.space.Volume = 5.0 gal
```

### FX Support
Currency Exchange Rates are used to define the conversion factors between currencies

```scala
scala> import squants.market.{CurrencyExchangeRate, Money}
import squants.market.{CurrencyExchangeRate, Money}

scala> // create an exchange rate
     | val rate1 = CurrencyExchangeRate(USD(1), JPY(100))
rate1: squants.market.CurrencyExchangeRate = USD/JPY 100.0

scala> // OR
     | val rate2 = USD / JPY(100)
rate2: squants.market.CurrencyExchangeRate = USD/JPY 100.0

scala> // OR
     | val rate3 = JPY(100) -> USD(1)
rate3: squants.market.CurrencyExchangeRate = USD/JPY 100.0

scala> // OR
     | val rate4 = JPY(100) toThe USD(1)
rate4: squants.market.CurrencyExchangeRate = USD/JPY 100.0

scala> val someYen: Money = JPY(350)
someYen: squants.market.Money = 350 JPY

scala> val someBucks: Money = USD(23.50)
someBucks: squants.market.Money = 23.50 USD

scala> // Use the convert method which automatically converts the money to the 'other' currency
     | val dollarAmount: Money = rate1.convert(someYen) // Money: 3.5 USD
dollarAmount: squants.market.Money = 3.50 USD

scala> val yenAmount: Money = rate1.convert(someBucks)  // Money: 2360 JPY
yenAmount: squants.market.Money = 2350 JPY

scala> // or just use the * operator in either direction (money * rate, or rate * money)
     | val dollarAmount2: Money = rate1 * someYen       // Money: 3.5 USD
dollarAmount2: squants.market.Money = 3.50 USD

scala> val yenAmount2: Money = someBucks * rate1		// Money: 2360 JPY
yenAmount2: squants.market.Money = 2350 JPY
```

### Money Context
A MoneyContext can be implicitly declared to define default settings and applicable exchange rates within its scope.
This allows your application to work with a default currency based on an application configuration or other dynamic source.
It also provides support for updating exchange rates and using those rates for automatic conversions between currencies.
The technique and frequency chosen for exchange rate updates is completely in control of the application.

```scala
import squants.market.{CAD, JPY, MXN, USD}
import squants.market.defaultMoneyContext

val exchangeRates = List(USD / CAD(1.05), USD / MXN(12.50), USD / JPY(100))
implicit val moneyContext = defaultMoneyContext withExchangeRates exchangeRates

val someMoney = Money(350) // 350 in the default Cur
val usdMoney: Money = someMoney in USD
val usdBigDecimal: BigDecimal = someMoney to USD
val yenCost: Money = (energyPrice * MegawattHours(5)) in JPY
val northAmericanSales: Money = (CAD(275) + USD(350) + MXN(290)) in USD
```

## Quantity Ranges
Used to represent a range of Quantity values between an upper and lower bound

```scala
scala> import squants.QuantityRange
import squants.QuantityRange

scala> val load1: Power = Kilowatts(1000)
load1: squants.energy.Power = 1000.0 kW

scala> val load2: Power = Kilowatts(5000)
load2: squants.energy.Power = 5000.0 kW

scala> val range: QuantityRange[Power] = QuantityRange(load1, load2)
range: squants.QuantityRange[squants.energy.Power] = QuantityRange(1000.0 kW,5000.0 kW)
```

Use multiplication and division to create a Seq of ranges from the original

```scala
// Create a Seq of 10 sequential ranges starting with the original and each the same size as the original
val rs1 = range * 10
// Create a Seq of 10 sequential ranges each 1/10th of the original size
val rs2 = range / 10
// Create a Seq of 10 sequential ranges each with a size of 400 kilowatts
val rs3 = range / Kilowatts(400)
```
Apply foreach, map and foldLeft/foldRight directly to QuantityRanges with a divisor

```scala
// foreach over each of the 400 kilometer ranges within the range
range.foreach(Kilometers(400)) {r => ???}
// map over each of 10 even parts of the range
range.map(10) {r => ???}
// fold over each 10 even parts of the range
range.foldLeft(10)(0) {(z, r) => ???}
```

NOTE - Because these implementations of foreach, map and fold* take a parameter (the divisor), these methods
are not directly compatible with Scala's for comprehensions.
To use in a for comprehension, apply the * or / operators as described above to create a Seq from the Range.

```scala
for {
    interval <- (0.seconds to 1.seconds) * 60  // 60 time ranges, 0s to 1s, 1s to 2s, ...., 59s to 60s
    ...
} yield ...
```

## Natural Language DSL
Implicit conversions give the DSL some features that allows user code to express quantities in a
more naturally expressive and readable way.

Create Quantities using Unit Of Measure Factory objects (no implicits required)

```scala
scala> import squants.market.Price
import squants.market.Price

scala> val load = Kilowatts(100)
load: squants.energy.Power = 100.0 kW

scala> val time = Hours(3.75)
time: squants.time.Time = 3.75 h

scala> val money = USD(112.50)
money: squants.market.Money = 112.50 USD

scala> val price = Price(money, MegawattHours(1))
price: squants.market.Price[squants.energy.Energy] = 112.50 USD/1.0 MWh
```

Create Quantities using Unit of Measure names and/or symbols (uses implicits)

```scala
scala> import scala.language.postfixOps
import scala.language.postfixOps

scala> import squants.energy.EnergyConversions._
import squants.energy.EnergyConversions._

scala> import squants.energy.PowerConversions._
import squants.energy.PowerConversions._

scala> import squants.market.MoneyConversions._
import squants.market.MoneyConversions._

scala> import squants.time.TimeConversions._
import squants.time.TimeConversions._

scala> val load1 = 100 kW 			        // Simple expressions don’t need dots
load1: squants.energy.Power = 100.0 kW

scala> val load2 = 100 megawatts
load2: squants.energy.Power = 100.0 MW

scala> val time = 3.hours + 45.minutes     // Compound expressions may need dots
time: squants.time.Time = 3.75 h
```

Create Quantities using operations between other Quantities

```scala
scala> val energyUsed = 100.kilowatts * (3.hours + 45.minutes)
energyUsed: squants.energy.Energy = 375000.0 Wh

scala> val price = 112.50.USD / 1.megawattHours
price: squants.market.Price[squants.energy.Energy] = 112.50 USD/1.0 MWh

scala> val speed = 55.miles / 1.hours
speed: squants.motion.Velocity = 24.587249174399997 m/s
```

Create Quantities using formatted Strings

```scala
scala> val load = Power("40 MW")		// 40 MW
load: scala.util.Try[squants.energy.Power] = Success(40.0 MW)
```

Create Quantities using Tuples

```scala
scala> val load = Power((40, "MW"))    // 40 MW
scala.MatchError: (40,MW) (of class scala.Tuple2)
  at squants.Dimension$class.parse(Dimension.scala:58)
  at squants.energy.Power$.parse(Power.scala:60)
  at squants.energy.Power$$anonfun$apply$1.apply(Power.scala:63)
  at squants.energy.Power$$anonfun$apply$1.apply(Power.scala:63)
  ... 1020 elided
```

Use single unit values to simplify expressions

```scala
scala> // Hours(1) == 1.hours == hour
     | val ramp = 100.kilowatts / hour
ramp: squants.energy.PowerRamp = 100000.0 W/h

scala> val speed = 100.kilometers / hour
speed: squants.motion.Velocity = 27.77777777777778 m/s

scala> // MegawattHours(1) == 1.megawattHours == megawattHour == MWh
     | val hi = 100.dollars / MWh
hi: squants.market.Price[squants.energy.Energy] = 100.00 USD/1.0 MWh

scala> val low = 40.dollars / megawattHour
low: squants.market.Price[squants.energy.Energy] = 40.00 USD/1.0 MWh
```

Implicit conversion support for using Double on the left side of operations

```scala
scala> val price = 10 / dollar	    // 1 USD / 10 ea
<console>:61: error: overloaded method value / with alternatives:
  (x: Double)Double <and>
  (x: Float)Float <and>
  (x: Long)Long <and>
  (x: Int)Int <and>
  (x: Char)Int <and>
  (x: Short)Int <and>
  (x: Byte)Int
 cannot be applied to (squants.market.Money)
       val price = 10 / dollar	    // 1 USD / 10 ea
                      ^
scala> val freq = 60 / second	    // 60 Hz
<console>:61: error: overloaded method value / with alternatives:
  (x: Double)Double <and>
  (x: Float)Float <and>
  (x: Long)Long <and>
  (x: Int)Int <and>
  (x: Char)Int <and>
  (x: Short)Int <and>
  (x: Byte)Int
 cannot be applied to (squants.time.Time)
       val freq = 60 / second	    // 60 Hz
                     ^
scala> val load = 10 * 4.MW		// 40 MW
load: squants.energy.Power = 40.0 MW
```

Create Quantity Ranges using `to` or `plusOrMinus` (`+-`) operators

```scala
val range1 = 1000.kW to 5000.kW	             // 1000.kW to 5000.kW
val range2 = 5000.kW plusOrMinus 1000.kW     // 4000.kW to 6000.kW
val range2 = 5000.kW +- 1000.kW              // 4000.kW to 6000.kW
```

### Numeric Support
Most Quantities that support implicit conversions also include an implicit Numeric object that can be imported
to your code where Numeric support is required.  These follow the following pattern:

```scala
scala> import squants.mass.{Grams, Kilograms} 
import squants.mass.{Grams, Kilograms}

scala> import squants.mass.MassConversions.MassNumeric
import squants.mass.MassConversions.MassNumeric

scala> val sum = List(Kilograms(100), Grams(34510)).sum
sum: squants.mass.Mass = 134510.0 g
```

NOTE - Because a quantity can not be multiplied by a like quantity and return a like quantity, the `Numeric.times`
operation of numeric is implemented to throw an UnsupportedOperationException for all types except `Dimensionless`.

The MoneyNumeric implementation is a bit different than the implementations for other quantity types
in a few important ways.

1. MoneyNumeric is a class, not an object like the others.
2. To create a MoneyNumeric value there must be an implicit MoneyContext in scope.
3. The MoneyContext must contain applicable exchange rates if you will be applying cross-currency Numeric ops.

The following code provides a basic example for creating a MoneyNumeric:

```scala
scala> import squants.market.MoneyConversions._
import squants.market.MoneyConversions._

scala> implicit val moneyContext = defaultMoneyContext
moneyContext: squants.market.MoneyContext = MoneyContext(squants.market.USD$@6fbc08a5,Set(squants.market.CLP$@43e3973f, squants.market.HKD$@407b0f33, squants.market.DKK$@49990bd2, squants.market.NZD$@76bf78fb, squants.market.SEK$@315ac8cf, squants.market.MYR$@6862add0, squants.market.CNY$@59dbd71, squants.market.GBP$@7e827626, squants.market.MXN$@43af3033, squants.market.EUR$@169c37b1, squants.market.BTC$@73a07b37, squants.market.KRW$@5bb5bee1, squants.market.CZK$@7ac891f4, squants.market.XAG$@6de8d42d, squants.market.ARS$@3b2f55ee, squants.market.INR$@413d4b75, squants.market.JPY$@36ebdd5c, squants.market.AUD$@731ac0dc, squants.market.NOK$@8b9cbd, squants.market.CHF$@66daa718, squants.market.USD$@6fbc08a5, squants.market.CAD$@279b1620, squants.market.BRL$@286ddaa8, squants.market.RUB$@...

scala> implicit val moneyNum = new MoneyNumeric()
moneyNum: squants.market.MoneyConversions.MoneyNumeric = squants.market.MoneyConversions$MoneyNumeric@6c0fe698

scala> val sum = List(USD(100), USD(10)).sum
sum: squants.market.Money = 110.00 USD
```

## Type Hierarchy
The type hierarchy includes the following core types:  Quantity, Dimension, and UnitOfMeasure

### Quantity and Dimension

A Dimension represents a type of Quantity. For example: Mass, Length, Time, etc.

A Quantity represents a dimensional value or measurement.  A Quantity is a combination of a numeric value and a unit.
For example:  2 lb, 10 km, 3.4 hr.

Squants has built in support for 54 quantity dimensions.

### Unit of Measure
UnitOfMeasure is the scale or multiplier in which the Quantity is being measured.
Squants has built in support for over 257 units of measure

For each Dimension a set of UOM objects implement a primary UOM trait typed to that Quantity.
The UOM objects define the unit symbols, conversion factors, and factory methods for creating Quantities in that unit.

### Quantity Implementations

The code for specific implementations include

* A class representing the Quantity including cross-dimensional operations
* A companion object representing the Dimension and set of available units
* A base trait for its Units
* A set of objects defining specific units, their symbols and conversion factors

This is an abbreviated example of how a Quantity type is constructed:

```scala
class Length(val value: Double, val unit: LengthUnit) extends Quantity[Length]  { ... }
object Length extends Dimension[Length]  { ... }
trait LengthUnit extends UnitOfMeasure[Length]  { ... }
object Meters extends LengthUnit { ... }
object Yards extends LengthUnit { ... }
```

The apply method of the UOM objects are implemented as factories for creating Quantity values.

```scala
val len1: Length = Meters(4.3)
val len2: Length = Yards(5)
```

Squants currently supports 257 units of measure

### Time Derivatives

Special traits are used to establish a time derivative relationship between quantities.

For example Velocity is the 1st Time Derivative of Length (Distance), Acceleration is the 2nd Time Derivative.

```scala
class Length( ... ) extends Quantity[Length] with TimeIntegral[Velocity]
...
class Velocity( ... ) extends Quantity[Velocity] with TimeDerivative[Length] with TimeIntegral[Acceleration]
...
class Acceleration( ... ) extends Quantity[Acceleration] with TimeDerivative[Velocity]
```

These traits provide operations with time operands which result in correct dimensional transformations.

```scala
scala> import squants.motion.{Acceleration, Velocity}
import squants.motion.{Acceleration, Velocity}

scala> import squants.space.Kilometers
import squants.space.Kilometers

scala> import squants.time.{Hours, Seconds, Time}
import squants.time.{Hours, Seconds, Time}

scala> val distance: Length = Kilometers(100)
distance: squants.space.Length = 100.0 km

scala> val time: Time = Hours(2)
time: squants.time.Time = 2.0 h

scala> val velocity: Velocity = distance / time
velocity: squants.motion.Velocity = 13.88888888888889 m/s

scala> val acc: Acceleration = velocity / Seconds(1)
acc: squants.motion.Acceleration = 13.88888888888889 m/s²

scala> val gravity = 32.feet / second.squared
gravity: squants.Acceleration = 9.7536195072 m/s²
```

Power is the 1st Time Derivative of Energy, PowerRamp is the 2nd.
```scala
scala> val power = Kilowatts(100)
power: squants.energy.Power = 100.0 kW

scala> val time: Time = Hours(2)
time: squants.time.Time = 2.0 h

scala> val energy = power * time
energy: squants.energy.Energy = 200000.0 Wh

scala> val ramp = Kilowatts(50) / Hours(1)
ramp: squants.energy.PowerRamp = 50000.0 W/h
```

## Use Cases

### Dimensional Analysis

The primary use case for Squants, as described above, is to produce code that is typesafe within domains
that perform dimensional analysis.

```scala
scala> import squants.mass.{Density, Mass}
import squants.mass.{Density, Mass}

scala> import squants.motion.VolumeFlow
import squants.motion.VolumeFlow

scala> import squants.space.VolumeConversions._
import squants.space.VolumeConversions._

scala> val energyPrice: Price[Energy] = 45.25.money / megawattHour
energyPrice: squants.market.Price[squants.energy.Energy] = 45.25 USD/1.0 MWh

scala> val energyUsage: Energy = 345.kilowatts * 5.4.hours
energyUsage: squants.energy.Energy = 1863000.0000000002 Wh

scala> val energyCost: Money = energyPrice * energyUsage
energyCost: squants.market.Money = 84.30 USD

scala> val dodgeViper: Acceleration = 60.miles / hour / 3.9.seconds
dodgeViper: squants.motion.Acceleration = 6.877552216615386 m/s²

scala> val speedAfter5Seconds: Velocity = dodgeViper * 5.seconds
speedAfter5Seconds: squants.motion.Velocity = 34.38776108307693 m/s

scala> val timeTo100MPH: Time = 100.miles / hour / dodgeViper
timeTo100MPH: squants.time.Time = 6.499999999999999 s

scala> val density: Density = 1200.kilograms / cubicMeter
density: squants.mass.Density = 1200.0 kg/m³

scala> val volFlowRate: VolumeFlow = 10.gallons / minute
volFlowRate: squants.motion.VolumeFlow = 6.30901964E-4 m³/s

scala> val flowTime: Time = 30.minutes
flowTime: squants.time.Time = 30.0 m

scala> val totalMassFlow: Mass = volFlowRate * flowTime * density
totalMassFlow: squants.mass.Mass = 1362.7482422399999 kg
```

### Domain Modeling
Another excellent use case for Squants is stronger typing for fields in your domain model.
This is OK ...

```scala
case class Generator(
  id: String,
  maxLoadKW: Double,
  rampRateKWph: Double,
  operatingCostPerMWh: Double,
  currency: String,
  maintenanceTimeHours: Double)

val gen1 = Generator("Gen1", 5000, 7500, 75.4, "USD", 1.5)
val gen2 = Generator("Gen2", 100, 250, 2944.5, "JPY", 0.5)
```

... but this is much better

```scala
import squants.energy.PowerRamp
import squants.energy.PowerRampConversions._

case class Generator(
  id: String,
  maxLoad: Power,
  rampRate: PowerRamp,
  operatingCost: Price[Energy],
  maintenanceTime: Time)

val gen1 = Generator("Gen1", 5 MW, 7.5.MW/hour, 75.4.USD/MWh, 1.5 hours)
val gen2 = Generator("Gen2", 100 kW, 250 kWph, 2944.5.JPY/MWh, 30 minutes)
```

### Anticorruption Layers

Create wrappers around external services that use basic types to represent quantities.
Your application code then uses the ACL to communicate with that system thus eliminating the need to deal
with type and scale conversions in multiple places throughout your application logic.

```scala
class ScadaServiceAnticorruption(val service: ScadaService) {
  // ScadaService returns meter load as Double representing Megawatts
  def getLoad: Power = Megawatts(service.getLoad(meterId))
  }
  // ScadaService.sendTempBias requires a Double representing Fahrenheit
  def sendTempBias(temp: Temperature) =
    service.sendTempBias(temp.to(Fahrenheit))
}
```

Implement the ACL as a trait and mix in to the application's services where needed.

```scala
import squants.radio.{Irradiance, WattsPerSquareMeter}
import squants.thermal.{Celsius, Temperature}

trait WeatherServiceAntiCorruption {
  val service: WeatherService
  def getTemperature: Temperature = Celsius(service.getTemperature)
  def getIrradiance: Irradiance = WattsPerSquareMeter(service.getIrradiance)
}
```

Extend the pattern to provide multi-currency support

```scala
class MarketServiceAnticorruption(val service: MarketService)
     (implicit val moneyContext: = MoneyContext) {

  // MarketService.getPrice returns a Double representing $/MegawattHour
  def getPrice: Price[Energy] =
    (USD(service.getPrice) in moneyContext.defaultCurrency) / megawattHour

  // MarketService.sendBid requires a Double representing $/MegawattHour
  // and another Double representing the max amount of energy in MegawattHours
  def sendBid(bid: Price[Energy], limit: Energy) =
    service.sendBid((bid * megawattHour) to USD, limit to MegawattHours)
}
```

Build Anticorruption into Akka routers

```scala
// LoadReading message used within a Squants enabled application context
case class LoadReading(meterId: String, time: Long, load: Power)
class ScadaLoadListener(router: Router) extends Actor {
  def receive = {
   // ScadaLoadReading - from an external service - sends load as a string
   // eg, “10.3 MW”, “345 kW”
   case msg @ ScadaLoadReading(meterId, time, loadString) ⇒
    // Parse the string and on success emit the Squants enabled event to routees
    Power(loadString) match {
      case Success(p) => router.route(LoadReading(meterId, time, p), sender())
      case Failure(e) => // react to QuantityStringParseException
    }
  }
}
```

... and REST API's with contracts that require basic types

```scala
trait LoadRoute extends HttpService {
  def repo: LoadRepository
  val loadRoute = {
    path("meter-reading") {
      // REST API contract requires load value and units in different fields
      // Units are string values that may be 'kW' or 'MW'
      post {
        parameters(meterId, time, loadDouble, unit) { (meterId, time, loadDouble, unit) =>
          complete {
            val load = unit match {
              case "kW" => Kilowatts(loadDouble)
              case "MW" => Megawatts(loadDouble)
            }
            repo.saveLoad(meterId, time, load)
          }
        }
      } ~
      // REST API contract requires load returned as a number representing megawatts
      get {
        parameters(meterId, time) { (meterId, time) =>
          complete {
            repo.getLoad(meterId, time) to Megawatts
          }
        }
      }
    }
  }
}
```

## Contributors

* Gary Keorkunian ([garyKeorkunian](https://github.com/garyKeorkunian))
* Jeremy Apthorp ([nornagon](https://github.com/nornagon))
* Steve Barham ([stevebarham](https://github.com/stevebarham))
* Derek Morr ([derekmorr](https://github.com/derekmorr))
* rmihael ([rmihael](https://github.com/rmihael))
* Florian Nussberger ([fnussber](https://github.com/fnussber))
* Ajay Chandran ([ajaychandran](https://github.com/ajaychandran))
* Gia Bảo ([giabao](https://github.com/giabao))
* Josh Lemer ([joshlemer](https://github.com/joshlemer))
* Dave DeCarpio ([DaveDeCaprio](https://github.com/DaveDeCaprio))
* Carlos Quiroz ([cquiroz](https://github.com/cquiroz))
* Szabolcs Berecz ([khernyo](https://github.com/khernyo))
* Matt Hicks ([darkfrog26](https://github.com/darkfrog26))
* golem131 ([golem131](https://github.com/golem131))

## Code of Conduct

Squants is a [Typelevel](http://typelevel.org/) Incubator Project and, as such, supports the Typelevel Code of Conduct.

## Caveats

Code is offered as-is, with no implied warranty of any kind.
Comments, criticisms, and/or praise are welcome, especially from scientists, engineers and the like.

# Release procedure

Making a release requires permission to publish to sonatype, and a properly setup [signing key](http://www.scala-sbt.org/sbt-pgp/usage.html):

To make a release do the following:

* Ensure the version is not set to `SNAPSHOT`

* Publish a cross-version signed package
```
  sbt +publishSigned
```

* Then make a release (Note: after this step the release cannot be replaced)
```
  sbt sonatypeRelease
```
