+++
title = "Entity Framework"
description = ""
weight = 1
alwaysopen = true
+++
#### Entities
Every table in your database that you want to be exposed to Entity Framework needs a class for it.

AbstractEntityFrameworkPOCO is an empty abstract class that can be used to add fields to every table in the database like TenantId.
```
namespace AdventureWorksNS.Api.DataAccess
{
	[Table("ContactType", Schema="Person")]
	public partial class ContactType: AbstractEntityFrameworkPOCO
	{
		public ContactType()
		{}

		public void SetProperties(
			int contactTypeID,
			DateTime modifiedDate,
			string name)
		{
			this.ContactTypeID = contactTypeID.ToInt();
			this.ModifiedDate = modifiedDate.ToDateTime();
			this.Name = name;
		}

		[Key]
		[Column("ContactTypeID")]
		public int ContactTypeID { get; set; }

		[Column("ModifiedDate")]
		public DateTime ModifiedDate { get; set; }

		[Column("Name")]
		public string Name { get; set; }
	}
}
```

#### Related Entities
You can include related entities by using .Include on the EF query.

```
return this.Context.Set<Customer>().Where(predicate).AsQueryable().OrderBy(orderClause).Skip(skip).Take(take).ToList<Customer>();
```
Changes to
```
return this.Context.Set<Customer>().Include(st => st.SalesTerritory.Name).Where(predicate).AsQueryable().OrderBy(orderClause).Skip(skip).Take(take).ToList<Customer>();
```

Then you have to modify your POCO and the object mapper to map this field to your POCO. 

#### Unit of Work
Unit of work is accomplished by passing a transaction manager to all controllers and starting a transaction in the request pipeline and then rolling it back if there are errors or committing if there are none.
```
public class EntityFrameworkTransactionCoordinator : ITransactionCoordinator
{
	private DbContext context;

	public EntityFrameworkTransactionCoordinator(DbContext context)
	{
		this.context = context;
	}

	public void BeginTransaction()
	{
		this.context.Database.BeginTransaction();
	}

	public void CommitTransaction()
	{
		if (this.context.Database.CurrentTransaction != null)
		{
			this.context.Database.CommitTransaction();
		}
	}

	public void DisableChangeTracking()
	{
		this.context.ChangeTracker.AutoDetectChangesEnabled = false;
	}

	public void EnableChangeTracking()
	{
		this.context.ChangeTracker.AutoDetectChangesEnabled = true;
	}

	public void RollbackTransaction()
	{
		if (this.context.Database.CurrentTransaction != null)
		{
			this.context.Database.RollbackTransaction();
		}
	}
}
```
Then in the filter that is applied to any methods you want unit of work applied to

```
public class UnitOfWorkAttribute : ActionFilterAttribute
{
	public override void OnActionExecuting(ActionExecutingContext actionContext)
	{
		if(!(actionContext.Controller is AbstractApiController))
		{
			throw new Exception("UnitOfWorkActionFilter can only be applied to controllers that inherit from AbstractApiController");
		}

		AbstractApiController controller = (AbstractApiController)actionContext.Controller;
		controller.TransactionCooordinator.BeginTransaction();
		
	}

	public override void OnActionExecuted(ActionExecutedContext actionExecutedContext)
	{
		AbstractApiController controller = (AbstractApiController)actionExecutedContext.Controller;

		if (actionExecutedContext.Exception == null)
		{
			try
			{
				controller.TransactionCooordinator.CommitTransaction();
			}
			catch (DbUpdateConcurrencyException)
			{
				throw;
			}
			catch (Exception)
			{
				throw;
			}
		}
		else
		{
			try
			{
				controller.TransactionCooordinator.RollbackTransaction();
			}
			catch (Exception)
			{
				throw;
			}
		}

		base.OnActionExecuted(actionExecutedContext);
	}
}
```

#### Concurrency
Concurrency is accomplished by applying an attribute to your Entity Framework POCO called ConcurrencyCheck. This attribute can be applied to multiple columns.

```
[Column("ModifiedDate")]
[ConcurrencyCheck]
public DateTime ModifiedDate { get; set; }
```
		
Typically you would use a column called rowversion which is usually a timestamp column. When you retrieve data the column is sent along with the data to the client.
When the client makes an update the value is sent and if it has changed in the database since the retrieve then Entity Framework will throw a DbUpdateConcurrencyException exception. You can catch this exception in a filter and return something to the client to tell them to retry or to reload the data. 

In the UnitOfWorkActionFilter we throw the exception by default. If you need retry functionality you will have to implement it
```
public override void OnActionExecuted(ActionExecutedContext actionExecutedContext)
{
	AbstractApiController controller = (AbstractApiController)actionExecutedContext.Controller;

	if (actionExecutedContext.Exception == null)
	{
		try
		{
			controller.TransactionCooordinator.CommitTransaction();
		}
		catch (DbUpdateConcurrencyException)
		{
			throw;
		}
		catch (Exception)
		{
			throw;
		}
	}
```


#### Read only requests
The ReadOnlyAttribute can be applied to any controller methods that only return data. This disables change tracking on the Entity Framework context which
drastically increases performance.
```
public class ReadOnlyAttribute : ActionFilterAttribute
{
	public override void OnActionExecuting(ActionExecutingContext actionContext)
	{
		if(!(actionContext.Controller is AbstractApiController))
		{
			throw new Exception("ReadOnlyFilter can only be applied to controllers that inherit from AbstractApiController");
		}

		AbstractApiController controller = (AbstractApiController)actionContext.Controller;
		controller.TransactionCooordinator.DisableChangeTracking();
	}

	public override void OnActionExecuted(ActionExecutedContext actionExecutedContext)
	{
		base.OnActionExecuted(actionExecutedContext);
	}
}
```

#### Spatial types
We do not currently support spatial types. Entity Framework doesn't support them and we set them to do not generate. When support 
is added to Entity Framework we will support them.

#### Primary Keys
We recommend unique, single field, integer primary keys. There is some contention on this issue but the reasons to design this way outweigh the negatives.

1. Storage cost is lower because primary keys are stored in any indexes and foreign keys.
2. Joins with composite keys are nasty.
3. Auto increment is built in.
4. An integer is guaranteed unique. A natural key often starts off unique but this can cause problems later when you realize SSN isn't really unique. 
5. To make sure your table really is a relation a composite unique index can accomplish your uniqueness. Changing this index later is cheap.
Changing a primary key after the fact stinks. 
6. A single field primary key makes a lot of sense for a REST API. A composite key doesn't. Get(field1,field2,field3,field4)
7. They are developer friendly.
8. An alternative is a guid while using NEWSEQUENTIALID(). As a developer I hate dealing with guids in a database. If you need to be able to merge databases
later you can add a field to all of the tables called rowguid which is designed for this. Guids eat lots of disk space as primary keys.

#### Performance hints

1. Turn off change tracking for read-only requests.
2. Don't select more data than you need.
3. Beware using included columns.
4. Only return the columns you need. We don't do this by default and I think this is a premature optimization.
5. Keep the number of records in the context small by not reading a lot of records.




