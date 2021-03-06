<html>
 <head>
   <title>MyFK_Insert</title>
 </head>
 
 <body>
 <h1>MyFK_Insert</h1>
 <h2>Motivation</h2>
 <p>If you want to insert an object, which usese multiple tables in a MySQL database, you often use many
    SELECT and INSERT statements. This is not very complicated, but can lead to a bulk of code which all
    looks similar: SELECT, INSERT, SELECT, SELECT, INSERT, SELCT, INSERT, ... . 
    Complete OR-Mapping frameworks solve this problem, but are often too complicated or too big for
    a simple insert into multiple tables.</p>
 
 <h2>Description</h2>
 <p>MyFK_Insert is a PHP class which inserts Objects in multiple Tables into a MySQL-DB
 respecting the integrity given on foreign keys.
 This works in MySQL as long the user has the right to see the foreign key assigment - which is default.

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
                                        // the Connection can be achieved by "$fkIns.getConnection()"
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
                     // Works only if 
                     //    * 3 is an existing ID in Person
                     //    * Third Parameter is an Int
                     //    * fk_Place exists as foreign key
                     // can be userful, if you already know the ID (performance)
                     // BUT have a look at the discussion about a new method called "fkIns.registerFK(tbl, fk_attr, fk_value)
</pre>

<pre>
// Example 4
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Zürich"  );
  $fkIns.register("Person", "name"    , "Fred"    );
  $fkIns.fkInsert(); // Fred will be inserted using fk_Place 2, because "Zürich" is already in the List.
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
  $fkIns.fkInsert(); // ERROR, due to the fk_Place can not be evaluated!
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
  
  
<h2>fkInsert() Internal Steps:</h2>
  <p><tt>fkInsert()</tt> does the following steps.</p>
  <ul>
   <li>Collect all registered inserts (all registered inserts after the last<tt>reset()<tt> call. This is done using the  <tt>register()</tt>-Method).</li>
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
   <li>Sort the fk-graph so, that all foreign-keys are ID in former tables. 
   This is done by numbering all leafs (tables without foreign keys) with 1. 
   From this point, all tables only pointing (via FK) to leafs get ID 2.
   Then all tables only pointig to already numbered tables, get 1 + highest-refered-table-number.</li>
   
   <li>Sort the tables according to the above "graph"-Number.</li>
   <li>Generate an empty IDMap having the table as Key and the Primary-Key (ID) as value.</li>
   <li>Now do the following steps for each table:
     <ul>
       <li>Create a SELECT-statement with all given (given throhug register()-statements and foreign keys out of the ID Table.) attributes.
       If it finds something, the ID (primary key) is inserted into the IDMap.</li>
       <li>If (the above) SELECT does not find anything, create an insert statement (if no auto-increment is given, add 1 to the max of the tabels primary key).
       <li>The newly inserted ID (or found via select) is inserted into the IDMap.</li>
     </ul></li>
  </ul>
  
  <h2>Discussion</h2>
  <ul>
   <li>The above statements are not very performant.</li>
   <li>The statements should be encapsulated into a transaction (BEGIN TRANSACT, COMMIT); so an error in the keys
       can do a ROLLBACK.</li>
   <li>Probably a foreign Key insert with a known foreign key ID (for performance) is better done using a separate regirter like  $fkIns.registerFK("Person", "fk_Place",          3); instead of   $fkIns.register("Person", "fk_Place",          3);</li>
   <li>...</li>
  </ul>
  
  
  </body>
  </html>
