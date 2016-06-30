# PostGIS/CartoDB reference

### List all tables in CartoDB acccount
From [StackExchange](http://gis.stackexchange.com/questions/94066/get-list-of-all-tables-in-cartodb-account-with-sql-statement)

	SELECT * FROM CDB_UserTables()
	
	
### Drop tables
[SQL Drop Table](http://www.1keydata.com/sql/sqldrop.html)
	
The syntax for DROP TABLE is,

	DROP TABLE "table_name";

Drop Multiple Tables At The Same Time

It is possible to drop more than one table at a time. To do that, list the names of all the tables we wish to drop separated by comma after DROP TABLE. For example, if we want to drop the User_Details table and the Job_List table together, we can issue the following SQL statement:

	DROP TABLE User_Details, Job_List;


### Spatially Enable Database

	CREATE EXTENSION postgis;	

From [Spatially enabled Postgres database](http://gis.stackexchange.com/questions/29759/spatially-enabled-postgres-database)


## Merge Multiple Tables

[Merge Multiple Tables StackExchange](http://gis.stackexchange.com/questions/27402/merge-multiple-tables-into-a-new-table-in-postgis)

(Pre-flight-check: are attributes identical in all original tables? Is the geometry type exactly the same in all tables?)

You can either

create the (empty) table first, then use INSERT INTO...SELECT... FROM to get all the data from each of the original tables into the merged one.
Create the new table from one big UNION statement.

For 1 it might go:

	CREATE TABLE merged (id serial primary key, attrib1 integer, attrib2 varchar(15),....);
	SELECT AddGeometryColumn('merged','geom',<SRID>,'<FEATURE_TYPE>,'XY');
	INSERT INTO merged (attrib1, attrib2, ...., geom) SELECT attribA, attribB,...,geom FROM table_1;
	INSERT INTO merged (attrib1, attrib2, ...., geom) SELECT attribA, attribB,...,geom FROM table_2;

and so on...

For option 2:

	CREATE TABLE merged AS( 
	SELECT attribA, attribB,...,geom FROM table_1
	UNION 
	SELECT attribA, attribB,...,geom FROM table_2
	UNION
	.... 
	);
	SELECT Populate_Geometry_Columns('merged'::regclass);

### Select by Date Time Range
[Specific Time Range Query in SQL Server](http://stackoverflow.com/questions/885188/specific-time-range-query-in-sql-server)

	SELECT * FROM MyTable WHERE datecolumn > '3/1/2009' AND datecolumn <= '3/31/2009'

### Making Lines from Points
[http://blog.cleverelephant.ca/2015/03/making-lines-from-points.html](http://blog.cleverelephant.ca/2015/03/making-lines-from-points.html)

Suppose you have a GPS location table:

* gps_id: integer
* geom: geometry
* gps_time: timestamp
* gps_track_id: integer

You can get a correct set of lines from this collection of points with just this SQL:

	SELECT
	  gps_track_id,
	  ST_MakeLine(geom ORDER BY gps_time ASC) AS geom
	FROM gps_poinst
	GROUP BY gps_track_id

### Making Points from Lines

#### Split at pre-determined distance
[Create a point on a line after a specific distance from startpoint](http://gis.stackexchange.com/questions/122373/create-a-point-on-a-line-after-a-specific-distance-from-startpoint)

Your friend here is ST_LineInterpolatePoint. Look here for more information: http://postgis.net/docs/ST_Line_Interpolate_Point.html.

You have to compute the right fraction to get your 10 metres. So this is done with this little calculcation:

	10 metres = ST_Length(the_line) * fraction 
	fraction = 10 metres / ST_Length(the_line)
So the result is (like the other posts described):

	ST_LineInterpolatePoint(the_geom, 10 / ST_Length(the_line))
Watch out the coordinate system you use so the 10 is 10 metres indeed.

IMPORT/EXPORT GEODATA BETWEEN SPATIALITE (SQLITE), POSTGIS, SHAPEFILE,…
http://isticktoit.net/?p=740


#CartoDB
### Download CSV of data table

	<a href=
	    "http://username.cartodb.com/api/v2/sql?q=SELECT%20*%20FROM%20table_name&format=CSV&filename=export.csv">
	    Download CSV
	</a>
	
### Get column names

	http://sheehan-carto.cartodb.com/api/v2/sql?q=SELECT column_name FROM information_schema.columns WHERE table_name ='<TABLE_NAME>'; &api_key=API_KEY	

### Column names to SQL query

	
	from cartodb import CartoDBAPIKey, CartoDBException, FileImport
	
	
	urlCols = "http://"+cartodb_domain+".cartodb.com/api/v2/sql?q=SELECT%20column_name%20FROM%20information_schema.columns%20WHERE%20table_name%20='"+tablename+"';%20&format=json&api_key="+API_KEY
	req = urllib2.Request(urlCols)
	response = urllib2.urlopen(req)
	cols_data = json.loads(str(response.read()))
	
	colsList = []
	for i in cols_data['rows']:
		colsList.append(i['column_name'])
	
	colsSQL = ',%20'.join(colsList) +',%20ST_X(the_geom),%20ST_Y(the_geom)'
	
	print colsSQL
	url = "http://"+cartodb_domain+".cartodb.com/api/v2/sql?q=SELECT%20"+colsSQL+"%20FROM%20"+tablename+"&format=csv&api_key="+API_KEY
	print url
	df = pd.read_csv(url)
	
	ouCSV = wp+'users_jitter.csv'
	#ouCSV = 'users.csv'
	

	df.to_csv(ouCSV, index=False)
	
	