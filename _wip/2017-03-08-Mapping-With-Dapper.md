---
layout: post
title: Deploying WebSites with MSDeploy and MSBuild
---

Getting Entities is easy. You don't even have to name your entity like your table.

```c#
	var holidays = _connection.Query<M_Holidays>($"SELECT * FROM {Table}");
```

Inserting Entities with plain Dapper is also easy, BUT very verbose if you have lots of fields in your table.

```c#
	var holiday = new M_Holiday(...);
	string sql = $@"
	    INSERT INTO
	        {Table}(
	        Pid, 
	        Code, Holiday, HolidayType, 
	        HolidayTypeXid, BrochureXid, 
	        BelongsTo, 
	        FromDate, ToDate, 
	        ChildFromAge, ChildToAge, AdultFromAge, AdultToAge, 
	        NoOfNights, NoOfElements, 
	        DepartsOn, MustInclude, MinNoPax, PerWhat, PNPH, DurationXid, CurrencyXid, 
	        Status, Taxable, VatCode, NightMultipleYN, RackRateYN, BuyHolidayXid, NotesXid, 
	        LastEdit, LastEditByXid, 
	        CompanyXid, FromTable, TourXid, SpecialOffer, GroundHandlerXid, MaxNoOfNights, MaxNoPax, 
	        TaxIndicator, SourceFrom, FlexibleHoliday, DynamicPackageYN, VatXid, Sales, AccuredSales, CostOfSale, AccuredCostOfSales, ExtensionPackageYN
	    ) values(
	        @Pid, 
	        @Code, @Holiday, @HolidayType, 
	        @HolidayTypeXid, @BrochureXid, 
	        @BelongsTo, 
	        @FromDate, @ToDate, 
	        @ChildFromAge, @ChildToAge, @AdultFromAge, @AdultToAge, 
	        @NoOfNights, @NoOfElements, 
	        @DepartsOn, @MustInclude, @MinNoPax, @PerWhat, @PNPH, @DurationXid, @CurrencyXid, 
	        @Status, @Taxable, @VatCode, @NightMultipleYN, @RackRateYN, @BuyHolidayXid, @NotesXid, 
	        @LastEdit, @LastEditByXid, 
	        @CompanyXid, @FromTable, @TourXid, @SpecialOffer, @GroundHandlerXid, @MaxNoOfNights, @MaxNoPax, 
	        @TaxIndicator, @SourceFrom, @FlexibleHoliday, @DynamicPackageYN, @VatXid, @Sales, @AccuredSales, @CostOfSale, @AccuredCostOfSales, @ExtensionPackageYN
	    );";
	_connection.Execute(sql, holiday);
```

And if we want to add custom parameters in addition to the entity:
[...]
 
With the Dapper.Contrib library that shortens to:

```c#
	var holiday = new M_Holiday(...);
	_connection.Insert(holiday);
```

But now your table has to be named M_Holiday.

You can create custom type handlers for "complex" types such as ids, which work for getting and inserting entities:

```c#

    public class UserIdHandler : SqlMapper.TypeHandler<UserId>
    {
        public override void SetValue(IDbDataParameter parameter, UserId value)
        {
            parameter.Value = value.Id;
        }

        public override UserId Parse(object value)
        {
            return new UserId((int)value);
        }
    }
```

If you want to have an enum in an entity, that's possible out of the box. However if you want it represented in a special way, that's where stuff gets tricky. 

Suppose you have an enum:
```c#
public enum YN
    {
        Yes = 'Y', No = 'N'
    }
```

That won't work with Dapper without a type handler:

```c#

    public class HolidayTypeHandler : SqlMapper.TypeHandler<HolidayType>
    {
        public override void SetValue(IDbDataParameter parameter, HolidayType value)
        {
            parameter.Value = value.ToString();
        }

        public override HolidayType Parse(object value)
        {
            return ((string)value)[0].ToEnum<HolidayType>();
        }
    }
```

And if you want the enum represented as a char in your Database, that won't work at all. Strings are fine, so is byte and int. 


## Conclusion

Dapper is great when you want to create Pocos for your Database objects. It is fast, has only a few functions and is easy to learn. It relies on sql text which you need read anyway. BUT it ties your entities directly to the database and if you want to have business entities, that's when stuff breaks down.
