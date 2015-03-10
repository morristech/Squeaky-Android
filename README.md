# Squeaky for Android

SQLite is simple and lightweight; it follows that managing SQLite databases on Android should be also.

Squeaky strives to be a straightforward approach to creating, migrating, and accessing SQLite
databases.

## Installation

### Build with Gradle

#### Prerequesites

* Gradle
* Java JDK 1.7
* Android SDK v21

#### Steps

1. Clone this repository to your machine.
1. `cd Android-Squeaky`
1. `gradle build`
1. `cp build/output/aar/Squeaky-release.aar [your project's lib folder]`

## Getting Started

### Define your Tables

    import com.bandcamp.squeaky.Table

    public class TodosTable extends Table {
        @Override
        public String getName() {
            return "todos";
        }

        @Override
        public int getVersion() {
            return 1;
        }

        @Override
        public String[] getCreateTable() {
            return new String[] {
                "CREATE TABLE todos ("+
                "    name TEXT NOT NULL"+
                "    completed INTEGER NOT NULL DEFAULT 0"+
                ")"
            };
        }

        @Override
        public String[] getMigration(int nextVersion) {
            return new String[0];
        }
    }

### Create your Database and add Tables to it

    // in your Application's or Activity's onCreate() method
    Database myDB = new Database(this, "todos_db");

    myDb.addTable(new TodosTable());
    myDb.prepare();

### Use it!

    myDb.insert("INSERT INTO todos (name) VALUES (?)", "Learn how to use Squeaky");
    myDb.insert("INSERT INTO todos (name, completed) VALUES (?, ?)", "Already done", 1);

    // query
    Cursor c = myDb.query("SELECT rowid, name, completed FROM todos");
    while (c.moveToNext()) {
        // get values
        long rowid = c.getLong(0);
        String name = c.getString(1);
        boolean completed = c.getInt(2) == 1;
    }

    // in an OnClickListener, for example..
    myDb.update("UPDATE todos SET completed = 1 WHERE rowid = ?", rowid);

### Migrate a Table

Modify your Table definition class:

    import com.bandcamp.squeaky.Table

    public class TodosTable extends Table {
        @Override
        public String getName() {
            return "todos";
        }

        @Override
        public int getVersion() {
            return 2;
        }

        @Override
        public String[] getCreateTable() {
            return new String[] {
                "CREATE TABLE todos ("+
                "    name TEXT NOT NULL"+
                "    completed INTEGER NOT NULL DEFAULT 0"+
                "    due_date INTEGER"+
                ")"
            };
        }

        @Override
        public String[] getMigration(int nextVersion) {
            if (nextVersion == 2) {
                return new String[] {
                    "ALTER TABLE todos ADD COLUMN due_date INTEGER"
                };
            }
            return new String[0];
        }
    }

That's it. Next time your database is prepared after adding the table definition, the table will be
migrated for you!