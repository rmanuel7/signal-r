# [Tutorial: Get started with ASP.NET Core SignalR](https://learn.microsoft.com/en-us/aspnet/core/tutorials/signalr?view=aspnetcore-9.0&WT.mc_id=dotnet-35129-website&tabs=visual-studio-code)

This tutorial teaches the basics of building a real-time app using SignalR. You learn how to:

&#x2714; Create a web project.<br>
&#x2714; Add the SignalR client library.<br>
&#x2714; Create a SignalR hub.<br>
&#x2714; Configure the project to use SignalR.<br>
&#x2714; Add code that sends messages from any client to all connected clients.<br>

---

## Create a web app project
```powershell
dotnet new webapp -o SignalRChat
```

---

## Add the SignalR client library
The SignalR server library is included in the ASP.NET Core shared framework. The JavaScript client library isn't automatically included in the project. For this tutorial, use Library Manager (LibMan) to get the client library from [unpkg](https://unpkg.com/). `unpkg` is a fast, global content delivery network for everything on [npm](https://www.npmjs.com/).

```powershell
libman install @microsoft/signalr@latest --provider unpkg --destination wwwroot/js/signalr --files dist/browser/signalr.js
```

The parameters specify the following options:

- Use the `unpkg` provider.
- Copy files to the `wwwroot/js/signalr` destination.
- Copy only the specified files.

The output looks similar to the following:

```powershell
Downloading file https://unpkg.com/@microsoft/signalr@latest/dist/browser/signalr.js...
wwwroot/js/signalr/dist/browser/signalr.js written to disk
Installed library "@microsoft/signalr@latest" to "wwwroot/js/signalr"
```

---

## Create a SignalR hub

A **hub** is a class that serves as a high-level pipeline that handles client-server communication.

In the SignalRChat project folder, create a `Hubs` folder.

In the `Hubs` folder, create the `ChatHub` class with the following code:

```csharp
using Microsoft.AspNetCore.SignalR;

namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```

The `ChatHub` class inherits from the SignalR [Hub](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.signalr.hub) class. The `Hub` class manages connections, groups, and messaging.

The `SendMessage` method can be called by a connected client to send a message to all clients. JavaScript client code that calls the method is shown later in the tutorial. SignalR code is asynchronous to provide maximum scalability.


---

## Configure SignalR
The SignalR server must be configured to pass SignalR requests to SignalR. Add the following highlighted code to the Program.cs file.

```csharp
using SignalRChat.Hubs;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddSignalR();

var app = builder.Build();

...

app.MapRazorPages();
app.MapHub<ChatHub>("/chatHub");

app.Run();
```

The preceding highlighted code adds SignalR to the ASP.NET Core dependency injection and routing systems.


---

## Add SignalR client code

Replace the content in Pages/Index.cshtml with the following code:

```html
@page
<div class="container">
    <div class="row p-1">
        <div class="col-1">User</div>
        <div class="col-5"><input type="text" id="userInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-1">Message</div>
        <div class="col-5"><input type="text" class="w-100" id="messageInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-6 text-end">
            <input type="button" id="sendButton" value="Send Message" />
        </div>
    </div>
    <div class="row p-1">
        <div class="col-6">
            <hr />
        </div>
    </div>
    <div class="row p-1">
        <div class="col-6">
            <ul id="messagesList"></ul>
        </div>
    </div>
</div>
<script src="~/js/signalr/dist/browser/signalr.js"></script>
<script src="~/js/chat.js"></script>
```

The preceding markup:

- Creates text boxes and a submit button.
- Creates a list with `id="messagesList"` for displaying messages that are received from the SignalR hub.
- Includes script references to SignalR and the `chat.js` app code is created in the next step.


In the `wwwroot/js` folder, create a `chat.js` file with the following code:

```JavaScript
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable the send button until connection is established.
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    document.getElementById("messagesList").appendChild(li);
    // We can assign user-supplied strings to an element's textContent because it
    // is not interpreted as markup. If you're assigning in any other way, you 
    // should be aware of possible script injection concerns.
    li.textContent = `${user} says ${message}`;
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
```

The preceding JavaScript:

- Creates and starts a connection.
- Adds to the submit button a handler that sends messages to the hub.
- Adds to the connection object a handler that receives messages from the hub and adds them to the list.
