<html>
 <head><title>MyFK_Insert</title>
 </head>
 
 <body>
 <h1>MyFK_Insert</h1>
 
 <p>MyFK_Insert inserts Objects in multiple Tables respecting the integrity given on foreign keys. This works in MySQL as long the user has the right to see the foreign key assigment - which is default.

FKeynsert stands for "Foreign-Key-Insert". Which means that the foreign keys are automatically found and inserted.</p>

<h2>Examples</h2>
<p>Suppose that you have the following two tables (it works whith any number of tables):</p>


<h3>Table 1 : Place</h3>
<pre>
ID | name

 1 , Toronto
 2 , Zürich
 3 , Oslo
 4 , Paris
</pre>

<h3>Table 2 : Person</h3>
<pre>
ID | name      | fk_Place

 1 , John      ,        1
 2 , Diana     ,        2
 3 , William   ,        1

</pre>

<p>Now, you insert like this</p>

<pre>
// Example 1
  $fkIns = new FKeynsert(dbconnection); //  or new FKeynsert(db-name [, username, password]]);
  
  $fkIns.register("Place" , "name"    , "Toronto" );
  $fkIns.register("Person", "name"    , "John"    );
  $fkIns.fkInsert(); // nothing will be done, because John is already in Toronto.
</pre>

<pre>
// Example 2
  $fkIns.reset(); //  reuse the connection, if possible
  $fkIns.register("Place" , "name"    , "Rome"    ); 
  $fkIns.register("Person", "name"    , "Ann"     ); 
  $fkKns.fkInsert(); // place Rome does not exist, so it will be inserted and its foreign key is insertet in "Person".
</pre>

<pre>
// Example 3
  $fkIns.reset();
  $fkIns.register("Person", "name"    , "Abraham" );
  $fkIns.register("Person", "fk_Place",          3);
  $fkIns.fkInsert(); // Abraham will be inserted as living in "Oslo"
                     // This can be useful, if you show the places as "selction-List" storing the ID.

</pre>

<pre>
// Example 4
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Zürich"  );
  $fkIns.register("Person", "name"    , "Fred"    );
  $fkIns.fkInsert(); // Fred will be inserte using fk_Place 2, because "Zürich" is already in the List.
</pre>

<pre>
// Example 5
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Helsinki");
  $fkIns.fkInsert(); // only a place is inserted
</pre>

<pre>
// Example 6
  $fkIns.reset();
  $fkIns.register("Person", "name"    , "Harold"  );
  $fkIns.fkInsert(); // ERROR, due to the fk_Person can not be evaluated!
</pre>

<pre>
// Example 7
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Bukarest");
  $fkIns.register("Person", "name"    , "Kilroy"  );
  $fkIns.register("Person", "fk_Place",          7);
  $fkIns.fkInsert(); // Ok if, and only if Bukarest gets ID 7 (next free). If not, an error occures.
</pre>


<pre>
// Example 8
  // suppose the Table "Person" has a "firstName" and a "familyName" as attributes:
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Bern"    );
  $fkIns.register("Person", "firstName", "Carl"   );
  $fkIns.fkInsert(); // OK, if and only if "familyName" allows NULL-values.
  </pre>
  
  
  <h2>fkInsert()</h2>
  <p><tt>fkInsert()</tt> does the following steps.</p>
  <ul>
   <li>Collect all registered inserts (this is already done calling <tt>register()</tt>).</li>
   <li>Collect all relevant tables.</li>
   <li>Collect all foreign-key tables like this:
<pre>
SELECT `TABLE_NAME`, 
       `COLUMN_NAME` as ForeignKey,
       `REFERENCED_COLUMN_NAME` as PrimaryKey,
       `REFERENCED_TABLE_NAME` as OtherTable 
FROM `KEY_COLUMN_USAGE` 
WHERE `TABLE_SCHEMA` = "test"  
AND NOT (`REFERENCED_TABLE_NAME` IS NULL);
</pre> (where "test" ist the name of the db)

   </li>
   <li>Sort the fk-graph so, that all foreign-keys are ID is former tables. 
   This is done by numbering all leafs (tables without foreign keys) with 1. 
   From this point, all tables only pointing (via FK) to leafs get ID 2.
   Then all tables only pointig to already numbered tables, get 1 + highest-refered-table-number.</li>
   
   <li>Sort the tables according to the above "graph"-Number.</li>
   <li>Generate an empty IDMap having the table as Key and the Primary-Key (ID) as value.</li>
   <li>Now do the following steps for each table:
     <ul>
       <li>Create a select-staement with all given (given throhug register()-Staements and foreign keys out of the ID Table.) attributes.
       If it finds something, the ID (primary key) is inserted into the IDMap.</li>
       <li>If (the above) Select does not find anything, create an insert statement (if no auto-increment is given, add 1 to the max of the tabels primary key).
       <li>The newly inserted ID (or found via select) is inserted into the IDMap.</li>
     </ul></li>
  </ul>
  
  <h2>Discussion</h2>
  <ul>
   <li>The above statements are not very performant.</li>
   <li>The statements should be encapsulated into a transaction (BEGIN TRANSACT, COMMIT); so an error in the keys
       can do a ROLLBACK.</li>
   <li>...</li>
  </ul>
  
  
  </body>
  </html>
