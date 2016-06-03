USE [Base]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

/*

Leigh
December 2015

Procedure RunSP autodeploys a BASE procedure into any defined data environment.
RunSP takes the name of the BASE procedure as a paramenter


*/


CREATE PROC [dbo].[RunSP] 
(
	@sproc varchar (100) = NULL, --base procedure to be deployed
	@env varchar(10) = NULL -- data environment into which @sproc will be compiled and executed
)

AS

SET NOCOUNT ON
DECLARE	@sprocName varchar(200) = DB_NAME() + '.' + SCHEMA_NAME() + '.' + OBJECT_NAME (@@PROCID)
BEGIN TRY

	DECLARE 
		@db varchar(20) = NULL,
		@spObject varchar(50) = NULL,
		@def nvarchar(max),
		@sql nvarchar(max) 
		
	IF @sproc is NULL
	BEGIN
		PRINT 'Please provide the name of the stored procedure you wish to run.'
		RETURN
	END
	
	SET @env = isnull (@env, 'Prod')
	
	IF @env not in (
		'Prod',  
		'LD',
		'Stage', 
		'Test'
		
	)
	BEGIN
		--alert the user they have provided an invalid environment parameter
		PRINT 'Invalid parameter ' + @env + '. Valid parameters include Prod (Mobile), Stage, Test, LD or no parameter.'
		PRINT 'If no parameter is provided, the default environment is Prod (Mobile).'	
	END		
	
	SET @db = CASE 
		when @env = 'Prod' then 'Production'
		when @env = 'Stage' then 'Stage'
		when @env = 'Test' then 'Development'
		end

	PRINT 'Procedure name: ' + @sproc + '

'
	PRINT 'Environment: ' + @env + '

'
	PRINT 'SQL will be spawned from BASE database ''Base''

'
	PRINT 'SQL will be executed in ENV database ''' + @db + '''

'
	SET @spObject = @db + '.dbo.' + @sproc


	--drop procedure if it exists
	IF OBJECT_ID (@spObject) IS NOT NULL 
	BEGIN 
		SET @sql = 'EXEC ' + @db + '.dbo.sp_executesql N''DROP PROC ' + 'dbo.' + @sproc + ''''
		EXEC (@sql)
	END 
		
	--clone procedure from base 
	PRINT 'Cloning procedure ' + @sproc + ' from BASE SQL (db = Base) to run in ' + @env + ' environment (db = ' + @db + ').

'
	SET @sql = (SELECT s.definition from sys.objects o JOIN sys.sql_modules s on o.object_id = s.object_id WHERE name = @sproc)

	IF @sql is NULL
	BEGIN
		PRINT 'Base SQL for procedure ' + @sproc + ' is not found (db = Base). No SQL to clone.

'
		RETURN

	END
	SET @sql = REPLACE (@sql, '''', '''''')
	SET @sql = N'EXEC ' + @db + '.dbo.sp_executesql N''' + @sql + ''''
	EXEC (@sql)

	PRINT 'Executing procedure ' + @sproc + ' in ' + @env + ' environment (db = ' + @db + ').

'

	SET @sql = N'EXEC ' + @db + '.dbo.sp_executesql N''' + @sproc + ''''
	EXEC (@sql)
	
		
END TRY
BEGIN CATCH
	DECLARE @errorNumber int = ERROR_NUMBER(), @errorMsg VARCHAR(1000) = ERROR_MESSAGE()
	EXEC Meta.dbo.ErrorLog @errorNumber, @errorMsg
END CATCH








GO

