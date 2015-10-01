## Download and Parse Json object

Most of the connection between server and client are using Json format to transfert data.
In Android it's now forbidden to make *http request* on the main thread, before we had to implement an "AsyncTask" to run request but now the can directly use the **Volley** and **Jackson libraries** that will do all the job for us !

Documentations:
- Volley : https://developer.android.com/training/volley/simple.html
- Jackson : http://www.tutos-android.com/parsing-json-jackson-android
https://github.com/FasterXML/jackson-databind

We will first get the json from the server (using Volley) and then automatically convert this Json to a ObjectArray.

### 1) Setup the android project

#### a) add Internet permissions in the *AndroidManifest*

add thos two lines in the <manifest> tag :

```xml
<manifest (...)>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    (...)
</manifest>
```

#### b) add libraries

There is serveral way to add libraries (cf google) you can download them manually or do like this

You have to add two libraries in the *build.grable (Module App)* in Grable Scripts tree.
You have to add those lines in dependencies :
```xml
compile 'com.fasterxml.jackson.core:jackson-core:2.4.2'
compile 'com.fasterxml.jackson.core:jackson-annotations:2.4.0'
compile 'com.fasterxml.jackson.core:jackson-databind:2.4.2'
compile 'com.mcxiaoke.volley:library:1.0.19'
```

and add packagingOptions in android { } brackets :
```xml
packagingOptions {
    exclude 'META-INF/LICENSE'
    exclude 'META-INF/NOTICE'
}
```

So this is how the *build.grable* file look like :

*build.grable*
```xml
apply plugin: 'com.android.application'
android {
    (...)
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/NOTICE'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.0.0'
    compile 'com.fasterxml.jackson.core:jackson-core:2.4.2'
    compile 'com.fasterxml.jackson.core:jackson-annotations:2.4.0'
    compile 'com.fasterxml.jackson.core:jackson-databind:2.4.2'
    compile 'com.mcxiaoke.volley:library:1.0.19'
}
```
#### c) *Sync* the project
after each modification in this file, android is asking you to sync the project so that it can take thos changement in consideration.
When it's done you are ready to work with them :)


### 2) Transmitting Network Data Using Volley

in *MainActivity.java*
```java

// Instantiate the RequestQueue.
RequestQueue queue = Volley.newRequestQueue(this);
String url ="http://the_url_of_the_server_that_is_supposed_to_send_you_a_Json.com";

// Request a string response from the provided URL.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
      //if its works
        new Response.Listener<String>() {
        @Override
        public void onResponse(String response) {
            /* TODO */
            //Display the first 500 characters of the response string.
            mTextView.setText("Response is: "+ response.substring(0,500));
        },

        // if it does not work
        new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            /* TODO */
            mTextView.setText("That didn't work!");
        }
});

// Add the request to the RequestQueue.
queue.add(stringRequest);
```
you have one method that is called if everything went well and an other one if it didn't work.
In this example you have to set the URL to the server and implement those two methods and that's it !

You can see that the response the Json is in the object *response* (the attribute the the onResponse method).
So now we have to parse this json into an array of java objects.

note that the *mTextView* is a TextView already define in the main_activity.xml and matched.

### 3) Parse Json to Java Object Using Jackson

For example let's say, the Json Object is designed that way :
```json
{"employees":[
    {"id":"123", "lastName":"Doe"},
    {"id":"007", "lastName":"Smith"},
    {"id":"444", "lastName":"Jones"}
]}
```
so first we have to create a java class that will match perfectly those attributes. Let's do it :

*employee.java*
```java
public class Employee {
  //attributes have to have the same name with Json
  private int id;            
  private String lastName;

  @Override
    public String toString() {
        return "employee n°" + id + " : " + lastName;
    }
}
```

We are now ready to parse the Json. you just have to do like this :

```java
//use the ObjectMapper from org.codehaus.jackson.map.ObjectMapper
final ObjectMapper objectMapper = new ObjectMapper();
String jsonObect = response; //we are getting in from the onResponse in the *MainActivity.java*

try {
    List<Employee> employees = objectMapper.readValue(response, new TypeReference<List<Employee>>(){});
    mTextView.setText(employeesList);
}
catch (IOException e) {
    e.printStackTrace();
}

```

### 4) Final result for MainActivity.java

This is how does look the MainActivty.java when we download and then read the json :

```java
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.widget.TextView;

import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;
import com.android.volley.toolbox.Volley;

import org.codehaus.jackson.map.ObjectMapper;
import java.io.IOException;


public class MainActivity extends ActionBarActivity {


  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);

      final TextView mTextView = (TextView) findViewById(R.id.tv1);
      final ObjectMapper objectMapper = new ObjectMapper();

      // Instantiate the RequestQueue.
      RequestQueue queue = Volley.newRequestQueue(this);
      String url ="http://the_url_of_the_server_that_is_supposed_to_send_you_a_Json.com";

      // Request a string response from the provided URL.
      StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
              //if its works
              new Response.Listener<String>() {
              @Override
              public void onResponse(String response) {
                  try {
                          List<Employee> employees = objectMapper.readValue(response, new TypeReference<List<Employee>>(){});
                          mTextView.setText(employees);
                      }
                      catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              },
              // if it does not work
              new Response.ErrorListener() {
              @Override
              public void onErrorResponse(VolleyError error) {
                  mTextView.setText("That didn't work!");
              }
      });
      // Add the request to the RequestQueue.
      queue.add(stringRequest);

    }
}
```

### 5) Update coming soon ..

This is the easy way to do that kind of operatn in Android. But we still have to do some improvment to not have to call *RequestQueue* all the time for example, by using an Application class.
I will add this in the next update of this file.