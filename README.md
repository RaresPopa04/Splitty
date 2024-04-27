# Splitty application (group 1)

## How to run

To run our application is quite simple:

1. Clone the project locally
2. Open a terminal and navigate to the root directory of the project (where this README is situated).
3. Run the gradle task `gradlew server:bootRun` or `gradlew bootRun`. In a (linux) terminal `./gradlew server:bootRun` or `./gradlew bootRun`.
   - This will start a server by default on `localhost:8080`
4. Wait for the server to start fully. Otherwise the client cannot start and the screen will show an error dialog, stating that the client cannot connect to the server.
5. Run the gradle task `gradlew client:run` or `gradlew run`. In a (linux) terminal `./gradlew client:run` or `./gradlew run`
   - This will start a client that by default listens to `localhost:8080`

## How to use the application
Before you can use the application, make sure you have started the server and client in that order. The instructions on how to do that can be found [here](#how-to-run).

### Create and join event
After starting the application,
you will see a window with a text field and a create button next to it.
In order to create a new event you can type the name of the event and hit the create button.
This will create a new event and load you into the overview from which you can manage the event.

If instead you want to join an event,
you can fill the invite code in the textfield, next to the join button.
The Hexcode can be found on the [main event page](#copy-event-code).


If you want to join an event you have recently visited, you can select it from the list of events at the bottom of the page.
You do this by pressing the arrow next to the name of the event you want to view again.
If you press the cross, it will remove the event from the list.

### Main event page
After you have joined an event, you will see the main event page. This page has the following functionalities to offer.

#### Edit event name
To edit the event name, simply click on the edit button (pen icon) next to the event name. It opens a new page where you can edit and save the name.

#### Copy event code
To copy the event code, you can click on the clipboard button next to the HEX-code. This will copy the code in your system. You can paste and send this code to someone else, so they can [join the event](#create-and-join-event).

#### Manage Participants
To add a participant you can press the add participant button (icon of a person with a small plus) situated on the event overview page.
In this add participant page, you can fill in the details of the participant and add it to the event.
Note that only the name of the new participant is required. Other fields are optional.

To edit or remove a participant, press the edit participant button (pencil icon next to the add participant button).
You get to see a list of all participants. Here, you select the participant you want to edit.  
After clicking the corresponding edit button of the participant, you see the page where you can change the details of the participant. You can click on the pen icon to edit the field, after which you can click the ok-button to save the new information.  
You can also remove the participant with the remove participant button.

#### Manage expenses
To add an expense, press the add expense button on the event overview page.
Then fill in all the necessary information of the expense and hit "ok" to add the expense to the event.
After adding the expense you can see it in the expense list on the event overview page. If the expense does not show immediately, press the All button next to the From and Including buttons.

To edit, remove or see more information of an expense, click on the edit button (pen icon) next to the expense.
In the edit expense page, you can edit the fields of the expense and save it by using the save button. To remove the expense press the remove expense button.
By pressing cancel you can return to the event overview.

#### Settle debts
In order to settle debts, you have to go to the statistics page.
There, you can see the balance of each participant.
If it is positive it means the participant should still receive money from the group.
If it is negative the participant should still pay money to the group.

When someone who still has to pay to the group gives money to someone who should still receive money, it can be added in the same way as a regular expense. In this way all debts can be settled.

### Switch language
To switch the language of the application press the switch language button the globe icon. This will switch the language between Dutch and English.

### Change server URL
To change the server url, first go to the create and join event page. In the top right corner, you can see the server-button. Clicking on it will show a new window where you can insert a new url. After entering the new url, click the Ok button to continue the application with this new server.

## Admin functions

### login into the overview
If you are on the page where you can create or join events.
You can press the admin menu button, and it should show an option to login.
After pressing this button it show a modal where you can fill in the [password](#password) of the server.
After filling in the password, you are able to see the admin overview.

#### Password
To login, you need to obtain the password of the server. The password will be in the console output of the terminal you run the server in.
It should be highlighted in the terminal and will look similar to: `password: t1G@UjpBFo`.

#### Logout
In order to logout you can either select the logout menu item under Admin or press the back button.
This brings you back to the page where you can create or join an event.

### Explore event
To see all the details of the event, select the event in the admin overview. Then you can click on the open event-button, which shows you the event page as a client.

### Order events
To order an event, click the name property you want to order on in the table that shows all existing events.
By clicking onces it sorts ascending; clicking twice it sorts descending. Clicking thrice will remove the filter.
To order on a different property just click the name of the new property to order on. 

### Delete an event
To delete an event from the database, select it from the list of events and press the delete event button.

### Backups

#### Load a JSON dump
To load an event from a JSON file press the Load JSON button on the admin overview.
You will then be prompted to select the JSON file you want to load.
After selecting the file containing a JSON dump of the event you wish to load it shows up in the list events in the admin overview.

#### Save a JSON dump
It is also possible to save an event as a JSON dump, allowing to create a backup or transfer it to a different server.
To do this, select the event you wish to save. Then select the location and name of the file you wish to save to.
This creates or overwrites the file to contain a JSON dump of the selected event.

## Where to find some other requirements

### Longpolling
An example of longpolling serverside can be found in the following file: `server/api/ParticipantController.java`. The main code would be the following:
```java
private Map<Object, Consumer<Participant>> listeners = new HashMap<>();

//***code***

@GetMapping("/updates")
public DeferredResult<ResponseEntity<Participant>> getUpdates(@PathVariable("eventId") long eventId) {
   //implementation
}
```
while an example of the usage of this would be the `listeners.forEach((v,l)-> l.accept(participant));` in:
```java
@PutMapping("")
@ResponseBody
public ResponseEntity<Participant> createParticipant(
   @PathVariable("eventId") long eventId,
   @RequestBody Participant participant) {
      // code
      listeners.forEach((v,l)-> l.accept(participant));
      // code
   }
```
and in all other functions when a participant gets deleted/edited/added.

The registering for updates clientside with longpolling can be found in `client/utils/ServerUtils.java` with the following function:
```java
public void registerForUpdates(long eventId,Consumer<Participant> participantConsumer) {
   //implementation
}
```
and this function is called in `client/scenes/MainEventPageCtrl.java` for registering to updates in:
```java
private void registerToParticipantUpdates() {
   //implementation
}
```
### Websockets
An example of websockets can be found in `server/api/EventController.java` which detects a deletion:
```java
@MessageMapping("/events")
@SendTo("/topic/events")
public Event deleteEvent(Event event) {
   //implementation
}
```
The `server/WebsocketConfig.java` takes care of the routes for a websocket.

In the `client/utils/ServerUtils.java`, the following code registers the websockets messages:
```java
public <T> void registerForMessages(String dest,Class<T> type,Consumer<T> consumer){
   //implementation
}
```

and in the main event page `client/scenes/MainEventPageCtrl.java`, the function is called:
```java
@Override
public void initialize(URL location, ResourceBundle resources) {
   //code
   serverUtils.registerForMessages("/topic/events",Event.class,e->{
      //implmenetation
   });
   //code
}
      
```
#   S p l i t t y  
 