# Use Android Pre Built database 
### Access database from assets folder.

In this simple Android project, i have shown how to access database from android assets folder.

Background:
We can not directly write into the files of assets folder in your application as the resources folders are read-only. So what we can do is, copy the database from assets folder and move to storage. Then we can use the database as we need.

## Copy database to storage
To copy the database, we can create a helper class as this:

~~~~
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
 
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
 
public class AssetDatabaseOpenHelper {
 
    private Context context;
    private String DB_NAME;
 
    public AssetDatabaseOpenHelper(Context context, String database_name) {
        this.context = context;
        this.DB_NAME = database_name;
    }
 
    public SQLiteDatabase saveDatabase() {
        File dbFile = context.getDatabasePath(DB_NAME);
 
        if (!dbFile.exists()) {
            try {
                copyDatabase(dbFile);
            } catch (IOException e) {
                throw new RuntimeException("Error creating source database", e);
            }
        }
 
        return SQLiteDatabase.openDatabase(dbFile.getPath(), null, SQLiteDatabase.OPEN_READONLY);
    }
 
    private void copyDatabase(File dbFile) throws IOException {
        InputStream is = context.getAssets().open(DB_NAME);
        OutputStream os = new FileOutputStream(dbFile);
 
        byte[] buffer = new byte[1024];
        while (is.read(buffer) > 0) {
            os.write(buffer);
        }
 
        os.flush();
        os.close();
        is.close();
    }
 
}
~~~~

## Use the database

After copy the database, we can do any oparation as Android SQLiate oparaion. We will only read data from database, so we can write database helper class like this:
~~~~
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
 
 
public class DataBaseHelper extends SQLiteOpenHelper {
 
    public DataBaseHelper(Context context, String db_name) {
        super(context, db_name, null, 1);
 
    }
 
    @Override
    public void onCreate(SQLiteDatabase db) {
 
    }
 
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
 
    }
 
    public Cursor getAllData(String tableName) {
 
        SQLiteDatabase db = this.getWritableDatabase();
        Cursor res = db.rawQuery("select * from " + tableName, null);
 
        return res;
 
    }
}

~~~~

### Finally MainActivity.java:
~~~~
import android.database.Cursor;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
 
public class MainActivity extends AppCompatActivity {
    String DB_NAME = "hello.db";
    String TABLE_NAME = "name";
    DataBaseHelper myDBHelper;
    Button button;
    TextView result;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        // Copy database
        AssetDatabaseOpenHelper assetDatabaseOpenHelper = new AssetDatabaseOpenHelper(this, DB_NAME);
        assetDatabaseOpenHelper.saveDatabase();
 
        myDBHelper = new DataBaseHelper(this, DB_NAME);
 
 
        result = (TextView) findViewById(R.id.resultView);
        button = (Button) findViewById(R.id.btn_view);
 
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Cursor res = myDBHelper.getAllData(TABLE_NAME);
                if (res.getCount() == 0) {
 
                    result.setText("No Data found, sorry!");
 
                    return;
                } else {
                    StringBuffer buffer = new StringBuffer();
                    while (res.moveToNext()) {
                        buffer.append("Id: " + res.getString(0) + "\n");
                        buffer.append("Name: " + res.getString(1) + "\n");
                    }
                    result.setText(buffer);
 
                }
            }
        });
 
 
    }
}
~~~~
