# MongoDB Driver Integration

## What was changed
`server/hi-repository/System/Admin/databaseDrivers.properties` registered MongoDB
with a non-existent driver class (`mongoDb=...`, no `jdbc:` prefix) and a second,
dead, duplicate entry (`com.helical.mongodb.MongoJdbcDriver`). Neither corresponded
to a real class, so MongoDB never actually worked as a connection type even though
`MongoConnectionFactory.java` already contains logic that specifically recognizes
the driver class `mongodb.jdbc.MongoDriver`.

The fix registers MongoDB the same way every other database in this file is
registered (driver class = JDBC URL template + default port), using the driver
class name that `MongoConnectionFactory` already expects:

```properties
#MongoDB (DbSchema open-source Mongo JDBC driver - jar must be placed in System/Drivers).
#For Atlas/SRV connections use jdbc:mongodb+srv://{{hostName}}/{{database}} as the URL instead.
mongodb.jdbc.MongoDriver=jdbc:mongodb://{{hostName}}:{{port}}/{{database}},27017
```

No other application code needed to change — this plugs into the existing,
driver-agnostic connection framework (same mechanism used by Postgres, MySQL,
Oracle, etc.), so no existing functionality is affected.

## How to configure/use a MongoDB connection
1. Download the open-source Mongo JDBC driver jar from
   https://github.com/wise-coders/mongodb-jdbc-driver (or dbschema.com), and
   place it in `server/hi-repository/System/Drivers/` — the same folder that
   already holds `postgresql-42.7.8.jar`. It's picked up automatically at
   runtime (no rebuild needed) by the app's driver directory watcher.
2. Restart the application.
3. In the Admin > Data Sources UI, create a new connection and select the
   MongoDB driver (`mongodb.jdbc.MongoDriver`).
4. Enter host, port (default `27017`), and database name — the JDBC URL is
   built automatically from the template above. For MongoDB Atlas, use a
   custom URL in the `jdbc:mongodb+srv://{{hostName}}/{{database}}` form instead.
5. Save and test the connection.
