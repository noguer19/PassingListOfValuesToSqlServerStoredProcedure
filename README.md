# Passing list of values to SQL Server Stored Procedure
This repository is intended to teach you (or remind myself) one of the different ways to pass a list of values to a SQL Server stored procedure.

### The first step is creating a 'User-Defined Table Type'.

```sql
CREATE TYPE [dbo].[IdList] AS TABLE(
	[Id] [int] NOT NULL
)
```

In this example, for simplicity reasons we are creating a Type to receive a list of integer values in our Stored Procedure, but we could also create a Type to receive a list of different values, for example:

```sql
CREATE TYPE [dbo].[IdList] AS TABLE(
  Id int NOT NULL,
  Active bit NOT NULL,
  CountryCode int NOT NULL
)
```

### The second step is to grant de execute permissions to the desired user or role

```sql
grant execute on type::[dbo].[IdList] to theAppropriateRoleOrUser
```

### Once we have created the Type and granted the permissions to the role or user, we create the Stored Procedure, which receive as parameter a variable of the Type we created earlier

```sql
CREATE PROCEDURE dbo.GetRecordsGivenTheId
(
  @ListOfIds dbo.IdList readonly --Here we declare a variable named ListOdIds of the type we created earlier.
)
AS 
  BEGIN
	-- Here we will simply asume we want to return the list of records related to the Ids that we are getting in our @ListOfIds variable
	SELECT *
	FROM MyTable as M
	WHERE M.Id IN (SELECT Id FROM @ListOfIds) 
  END
```

### The way we pass the list of Ids to the stored procedure from C# for example is:

```c#
public List<ClassThatHoldsTheDataReturnedByTheStoredProcedure> GetRecordsByGivenIds(DataTable listOfIds)
        {
            var argList = new List<SqlParameter>();
            SqlParameter tab = new SqlParameter("@ListOfIds", listOfIds)
            {
                TypeName = "dbo.IdList"
            };
            argList.Add(tab);
            object[] prm = argList.ToArray();
            var csv = string.Join(",", argList.Select(l => l.ParameterName));
            var listOfRecordsById = _db.Database.SqlQuery<ClassThatHoldsTheDataReturnedByTheStoredProcedure>("dbo.GetRecordsGivenTheId " + csv, prm).ToList();
            return listOfRecordsById;
        }

```

Here we are assuming we are using Entity Framework and have an instance of a DbContext class named "_db"
