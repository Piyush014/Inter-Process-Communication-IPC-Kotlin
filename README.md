# Inter-Process-Communication-IPC-Kotlin


## Sequence Diagram For the Entire IPC Mechanism on Both the apps.

![mahakali-Server Workflow (1)](https://github.com/user-attachments/assets/a8b31f45-1805-45ae-86f8-051e978df7e3)


# Inter Process Communication:

IPC, or inter-process communication, refers to a general concept of communication between separate processes. In Android, IPC encompasses two specific scenarios:

1. Communication between different applications.
2. Communication between processes within a multi-process application, where components like Activities, Services, Broadcast Receivers, or Content Providers are running in separate processes.


### Android offers three primary methods for inter-process communication (IPC):
  1. AIDL (Android Interface Definition Language)
  2. Messenger
  3. Broadcast

 - Although traditional Linux-based techniques (like socket communication or shared files) can also be used for IPC, Android’s official documentation recommends choosing one of these three Android-specific methods for better security. These approaches offer advantages such as authenticating the application initiating the communication.

## Which method should you use? 
If you're unsure which IPC method is right for your situation, you can refer to the following decision-making chart for guidance.

![image](https://github.com/user-attachments/assets/08f288bb-6ac4-4d16-95bd-c8e645257a61)

# Android IPC Mechanisms #1 — AIDL:

You should opt for AIDL (Android Interface Definition Language) if your application requires a multithreaded IPC structure. If concurrent operations are unnecessary, you can consider using the Messenger method instead.

With AIDL, one application can invoke methods in another application on the same device, a concept known as Remote Procedure Call (RPC). Since methods can be called from different processes or threads simultaneously, it's crucial to ensure the mechanism handling these calls is thread-safe.


## What is AIDL?

AIDL stands for Android Interface Definition Language. If you're familiar with the concept of IPC, you can think of AIDL as Android's version of an Interface Definition Language (IDL). This method is called AIDL because the communication interface between applications or processes is established using the AIDL language. AIDL has a syntax similar to Java, making it easier for developers familiar with Java to use it for inter-process communication on Android.

### Let's implement a scenario with the following features:

- The client retrieves the server's process ID (PID) and the number of connection requests the server has received.
- The client obtains the total count of connection requests made to the server.
- The client sends its own information, such as its PID and package name, to the server application. The server then displays this information as the details of the last connected client on the UI.

In this method, communication relies on the Binder architecture. To initiate a service, the `bindService` method is used. A component within the application or a remote process can bind to the service, and a binder object is returned to the component (such as an activity or service) that binds to it. We won’t delve deeper into the concept of Bound Services here, as it’s beyond the scope of this article.

In this example, the client will bind to the service of the server application, allowing the client to access the methods defined in the server.


### Creating the Server Application

First, start by creating a new project. In this example, we'll name it **IPCServer**.

### Creating an AIDL File

Next, create an AIDL file that contains the methods the client will call. To do this, right-click on the project explorer (on the left side of the IDE) and follow the steps:

### `New -> AIDL -> AIDL File`

When you create the AIDL file this way, it will automatically be placed in the `aidl` folder, which is located alongside the `java` and `res` directories. For example, in this project, the path to the file will be `src/main/aidl/com/piyush/ipcserver/`.

**Modifying the AIDL File**

Now, let's modify the AIDL file to add custom methods instead of the default `basicTypes` method. Here's what each method will do:

1. **getPid**: Retrieves the process ID (PID) of the server application.
2. **getConnectionCount**: Returns the number of client connection requests the server has received during its lifecycle.
3. **setDisplayedValue**: The client sends information about itself, such as the package name, PID, and a custom message, to the server. The server will display this information in its GUI.

Here's how the modified AIDL file should look:

// IServerAidl.aidl
package com.piyush.ipcserver;

// Declare the AIDL interface
interface IServerAidl {
    // Method to get the process ID of the server application
    int getPid();

    // Method to get the total number of client connection requests
    int getConnectionCount();

    // Method for the client to send its package name, PID, and custom text
    void setDisplayedValue(String packageName, int pid, String message);
}

With this structure, the client can call these methods to retrieve and send data to the server, while the server handles and responds to the client's requests.


When working with AIDL, there are specific restrictions on the types of parameters and return values that methods can use. Here's a list of all supported types:

- **Primitive types**: All Java primitive types, such as `int`, `long`, `char`, `boolean`, etc.
- **Arrays of primitive types**: Such as `int[]`, `float[]`, etc.
- **String**: Standard Java `String` type.
- **CharSequence**: For handling sequences of characters.
- **List**: The elements in the `List` must be one of the supported data types or other AIDL-generated interfaces or parcelables. You can also use parameterized types (e.g., `List<String>`). The other side will always receive an `ArrayList`, but the method is generated using the `List` interface.
- **Map**: Similar to `List`, all elements in the `Map` must be supported types or AIDL-generated parcelables. However, parameterized maps (e.g., `Map<String, Integer>`) are **not supported**. On the other side, a `HashMap` is returned, though the method is generated with the `Map` interface. For more flexibility, you can consider using a `Bundle` instead of a `Map`.

If you want to use custom classes as parameters or return types, they need to implement the **Parcelable** interface. This ensures that Android can serialize and parse them into a form the operating system understands for IPC.

By adhering to these type restrictions, you can ensure that your AIDL methods can effectively handle communication between processes.


### Directional Tags: in, out, and inout

In AIDL, all non-primitive parameters require a directional tag to specify the flow of data: **in**, **out**, or **inout**.

- **in**: Indicates an input-only parameter. This parameter sends data from the client to the server but does not receive data back.
  
- **out**: Specifies an output-only parameter. This means that the parameter will not contain meaningful data on input but will be filled with data during the method execution. For example, a method that copies an array of bytes might be defined as:
  
  void copyArray(in byte[] source, out byte[] dest);
  

- **inout**: Indicates that the parameter has significance in both input and output contexts. It can both send data to the server and receive data back. For example:
  
  void charsToUpper(inout char[] chars);
  

These tags are crucial because every parameter must be marshalled (serialized, transmitted, received, and deserialized) for IPC. By using the **in**, **out**, and **inout** tags, the Binder can optimize the marshalling process, skipping unnecessary steps for parameters that don’t require it, thus improving performance.

Thanks to Perihan Mirkelam for this insightful explanation!

### Generating the Stub

Once you have created your AIDL file, follow these steps to build the project:

1. Go to **Build** in the menu.
2. Select **Rebuild Project**.

After rebuilding, you will notice that a file with the same name as your AIDL file, but with a `.java` extension, is automatically generated. This file contains an interface that includes an abstract subclass named `Stub`, which implements the methods you defined in the AIDL file. In the server's service, you will extend your binder object from this `Stub` class.

### Defining Data Classes

1. **Create the Client Data Class**: Start by defining a data class for the client.

2. **Create a Singleton RecentClient Class**: Next, implement a singleton class named `RecentClient` to store information about the last connected client. Although you could maintain a list of all connected clients, this example will focus on displaying only the details of the most recently connected client in the UI.

### Creating a Service

1. **Create the Service**: Begin by creating a service class.

2. **Implement the Binder Object**: Extend your service class from the `Stub` class to create your binder object. In this class, implement the methods defined in your AIDL interface.

3. **Return the Binder Object**: Use the `bindService` method to return your binder object to the client.

4. **Update RecentClient**: When a client binds to or unbinds from the service, update the `RecentClient` object to ensure the UI reflects the latest connection information.

**Note**: If you plan to perform concurrent operations within the methods of the binder object, consider using Coroutines for better handling of asynchronous tasks.



### Defining the Service in the Manifest

Ensure to add the service definition in your `AndroidManifest.xml` file. Here’s how to do it:

1. **Declare the Service**: Inside the `<application>` tag, define your service.

2. **Add an Intent Filter**: Include an intent filter to specify the action associated with the service. This will mark the service as exported by default, allowing it to be accessed from other processes.

Here's an example of how your manifest entry might look:

<service
    android:name=".YourServiceClass"
    android:exported="true">
    <intent-filter>
        <action android:name="com.piyush.ipcserver.ACTION" />
    </intent-filter>
</service>

Make sure to replace `YourServiceClass` with the actual name of your service class. The action name defined here will be used in the client application to interact with the service.
### Use ViewBinding
For simplicity in code management, we can choose to implement the ViewBinding library. Let's start by declaring our intent to use it.

### Modify the Activity
We will update the Activity to reflect client information, displaying details of the last connected client. If there is no connected client, the layout will remain hidden.

### Create the Client Application
1. **Start a New Project**: Create a new project and name it `IPCServer`.
2. **Choose a Template**: For the remainder of this article, we'll use this application for various IPC methods (like Broadcast and Messenger). Thus, select the "Bottom Navigation Activity" from the activity gallery when creating the project. This template will accommodate AIDL, Messenger, and Broadcast methods.

### Copy-Paste the AIDL File
1. **Add AIDL File**: Once the project is open, include the exact same AIDL file created in the server application and rebuild the project.
2. **Creating AIDL File**: To create the file, navigate to `New -> AIDL -> AIDL File`.
3. **Directory Structure**: It’s crucial that the directory structure for the AIDL file is consistent in both applications. When created via the IDE, it defaults to `src/main/aidl/com/piyush/ipcclient/`, but it should be updated to `src/main/aidl/com/piyush/ipcserver/`.
4. **Copy Content**: Make sure to copy the entire content of the AIDL file, including the package name.

### Set UI-Related Elements
1. **Edit Menu File**: Update the menu file located in the `/res/menu/` directory.
2. **Update Navigation File**: Modify the navigation file found in the `/res/navigation/` directory.
3. **Edit String Resources**: Make necessary changes to the string resource file.
4. **Change Theme Colors**: Adjust colors in the theme to visually differentiate it from the server.
5. **Leave Activity XML Unchanged**: Do not alter the Activity XML file.

### Configure Navigation and Binding in Activity
Set up the navigation and binding configuration in your Activity.

### Create a Fragment
1. **Fragment XML File**: Create the XML layout file for the fragment.
2. **Connect-Disconnect Button**: Add a button to facilitate connection and disconnection from the server.
3. **Hidden Layout**: Include a layout that will become visible upon establishing a connection. This layout will display information received from the server application.

### Implement Business Logic in Fragment
1. **View Binding**: Set up View Binding within the Fragment.
2. **Bind to Server**: When the Connect button is pressed, bind to the server service. If the service is not yet created during the bind process, use the `BIND_AUTO_CREATE` flag for automatic creation.
3. **Service Connection Listening**: Implement the `ServiceConnection` interface to listen for connection events.
4. **Send Messages**: Upon establishing the connection, send the package name and PID to the server, then display the responses in the GUI.
5. **Clean Up on Disconnection**: When the connection is broken, ensure to unbind from the service. Background services in applications without user interaction can be killed by the operating system to free up memory. In Bound Services, as long as at least one client is bound, background services remain active, except in extreme cases. Therefore, avoid remaining bound to the service when it's not in use.

### Finalization of the Client Application
Now that we've set everything up, the client application is complete. The flow in the graphical interface when invoking methods on the server will function as described.

### Messenger Overview
**What is Messenger?**  
Messenger is a Handler sent to a remote process, utilizing the Binder architecture akin to AIDL. Implementing Messenger is generally easier than AIDL since it creates and uses AIDL files in the background, freeing developers from that complexity. 

However, while Messenger offers convenience, it has limitations compared to AIDL. For instance, AIDL supports concurrent operations, while Messenger executes calls sequentially, ensuring that messages are processed from only one process or thread at a time.

### How Messenger Works
As with the previous example, the client binds to the server’s service, receiving a Messenger object in return. This setup enables communication between the two applications through Message objects.

### Begin with the Server Application
1. **Declare Intent Filter**: In the Manifest, add the action `messengerexample` that will be sent from the client application to the service. This action will specify which IPC method the client is using.

2. **Update the Service**: Create a Handler in the service to process messages from the client, updating the RecentClient object to refresh the GUI as necessary. Establish the Messenger with the Handler for communication.

3. **Handle Client Messages**: The `msg.replyTo` object from the client's message can be captured for potential responses. This facilitates two-way communication without much complexity.

4. **Update onBind Method**: Modify the `onBind` method in the service to accommodate clients binding with the `messengerexample` action. The Messenger object, containing an internal binder, will be returned to the client.

### Client Application Adjustments
1. **Add Constants**: Incorporate the constants used for bundle keys in the client application as well.
2. **Update XML**: Use the same XML layout for the fragment as used in the AIDL setup, with the only difference being the button name.

### Update the Fragment
1. **Define Messenger Objects**: Introduce two Messenger objects in the fragment:
   - `clientMessenger`: For sending messages to the server.
   - `serverMessenger`: For the server to reply back.

2. **Define Incoming Message Handler**: Create a Handler to manage incoming messages, updating the UI accordingly.
3. **Bind and Unbind**: Bind to the server upon clicking the connect button and unbind upon clicking the disconnect button. When successfully bound, send messages to the server.
4. **Message Handling**: Set `message.replyTo` to `clientMessenger` for expected responses. If the service connection is lost, clear the information displayed on the UI.

### Understanding Broadcasts
**Broadcasts** differ from the other IPC techniques as they can be sent by both the system and applications. Applications can register for specific broadcasts using `BroadcastReceiver`. 

Receivers are typically exported, allowing them to handle broadcasts from any other application. To add a security layer, permissions and action filters can be specified in the `<receiver>` tag in the Manifest. For this example, we will send broadcasts explicitly to particular applications.

1. **Explicit Receivers**: These are exempt from the restrictions applicable to Manifest-defined receivers for API level 26. When defined in the Manifest, receivers are active when the application is loaded and can launch the application even if it is not running upon receiving a broadcast.

### Client Application for Broadcast
1. **Sending Broadcast**: Since there is no two-way communication, the client will only send a broadcast with its information and display the broadcast time on the screen.
2. **Define Explicit Intent**: Create an explicit intent with the server's package name and receiver class name to ensure only specific applications can listen to the broadcast.

### Server Application for Broadcast
1. **Declare Receiver**: Mark the receiver as exported in the Manifest.
2. **Add Broadcast Filter**: Include the action filter utilized in the broadcast intent.
3. **Create Receiver Class**: Implement the receiver class and update the client information on the UI based on the received data.

### Conclusion
We have explored various interprocess communication methods in Android by developing client-server applications supporting three different IPC techniques. The source code for the complete application is available in this repo.

Special Thanks to @perihanmirkelam for their Insightful explaination.
