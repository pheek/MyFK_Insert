<html>
 <head><title>MyFK_Insert</title>
 </head>
 
 <body>
 <h1>MyFK_Insert</h1>
 
 <pre>


MyFK_Insert inserts Objects in multiple Tables respecting the integrity given on foreign keys. This works in MySQL as long the user has the right to see the foreign key assigment - which is default.

FKeynsert stands for "Foreign-Key-Insert". Which means that the foreign keys are automatically found and inserted.

Examples:
Suppose that you have the following two tables (it works whith any number of tables):


Table 1 : Place
***************
ID | name
----------------
 1 , Toronto
 2 , Zürich
 3 , Oslo
 4 , Paris
----------------


Table 2 : Person
****************
ID | name      | fk_Place
-------------------------
 1 , John      ,        1
 2 , Diana     ,        2
 3 , William   ,        1
-------------------------


Now, you insert like this

// Example 1
  $fkIns = new FKeynsert(dbconnection); //  or new FKeynsert(db-name [, username, password]]);
  
  $fkIns.register("Place" , "name"    , "Toronto" );
  $fkIns.register("Person", "name"    , "John"    );
  $fkIns.fkInsert(); // nothing will be done, because John is already in Toronto.

// Example 2
  $fkIns.reset(); //  reuse the connection, if possible
  $fkIns.register("Place" , "name"    , "Rome"    ); 
  $fkIns.register("Person", "name"    , "Ann"     ); 
  $fkKns.fkInsert(); // place Rome does not exist, so it will be inserted and its foreign key is insertet in "Person".

// Example 3
  $fkIns.reset();
  $fkIns.register("Person", "name"    , "Abraham" );
  $fkIns.register("Person", "fk_Place",          3);
  $fkIns.fkInsert(); // Abraham will be inserted as living in "Oslo"
                     // This can be useful, if you show the places as "selction-List" storing the ID.

// Example 4
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Zürich"  );
  $fkIns.register("Person", "name"    , "Fred"    );
  $fkIns.fkInsert(); // Fred will be inserte using fk_Place 2, because "Zürich" is already in the List.

// Example 5
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Helsinki");
  $fkIns.fkInsert(); // only a place is inserted

// Example 6
  $fkIns.reset();
  $fkIns.register("Person", "name"    , "Harold"  );
  $fkIns.fkInsert(); // ERROR, due to the fk_Person can not be evaluated!

// Example 7
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Bukarest");
  $fkIns.register("Person", "name"    , "Kilroy"  );
  $fkIns.register("Person", "fk_Place",          7);
  $fkIns.fkInsert(); // Ok if, and only if Bukarest gets ID 7 (next free). If not, an error occures.

// Example 8
  // suppose the Table "Person" has a "firstName" and a "familyName" as attributes:
  $fkIns.reset();
  $fkIns.register("Place" , "name"    , "Bern"    );
  $fkIns.register("Person", "firstName", "Carl"   );
  $fkIns.fkInsert(); // OK, if and only if "familyName" allows NULL-values.
  
  </pre>
  </body>
  </html>
