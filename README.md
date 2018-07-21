# ARK Observer
A simple macro library to use in ARK modding projects that provides Observer Pattern-esque functionality. This is not a true Observer Pattern implementation, but uses many of the same concepts.

This provides a more robust way to pass data between actors in a mod. This even works between different mods, which is useful because the actors using it don't need to have references to anything they are passing the data to.

## Useful Reading

**Observer Pattern:** [https://en.wikipedia.org/wiki/Observer_pattern](https://en.wikipedia.org/wiki/Observer_pattern)

# Setup Instructions

1. Clone the repository to the ARK Dev Kit's `mods` folder, or download the `ARKObserverMacroLibrary.uasset` file and place it somewhere within the ARK Dev Kit's `mods` folder.

# Usage Instructions

There are currently six macros made available by this library:
1. `EmitEvent` - emits an event that other actors can listen for. You need to specify an event name and any event data you wish to send.
2. `HandleEvent` - takes input from the `ActorCustomEvent` node (available in the ARK Dev Kit) and processes it, returning the event name and event data.
3. `GetData` - takes the event data (previously processed by `HandleEvent`) and a key and returns the event data that corresponds to that key.
4. `BuildEventString` - takes the event data and an event name and returns a string representing the event with its data. This happens automatically in `EmitEvent`, but I've made this method available in case you would like to get a raw event for some reason.
5. `BuildDataString` - takes a name and a value corresponding to the data you want to send and creates a data string using the name and value. If you don't want to use this method, you can easily create the data string yourself by putting the delimiter `:=:` between the name and the value (e.g. `PlayerName:=:MyPlayerName`). The `:=:` delimiter is to reduce the possibility of data containing characters that could conflict with the observer. **If you choose to create your data string manually, it is up to you to sanitize your data to make sure it doesn't include the following delimiters: `#$#`, `:=:`, `!@!` (used internally)**
6. `SanitizeEventData` - used internally in `BuildDataString`, this makes sure the data passed in is properly sanitized to not contain any of the internally used delimiters. It also trims whitespace.

## Emitting an Event

To emit an event, follow these steps ([click here for an example](https://github.com/GyozaGuy/ark-observer#emitting-an-event-1)):
1. Prepare the data you wish to send.
2. Find the `BuildDataString` macro and provide a name for the data and the associated data you wish to send.
3. Repeat step 2 for all data you want to add to the event.
4. Find the `EmitEvent` macro and enter an event name directly into the appropriate input.
5. Drag off of the `Event Data` input on the `EmitEvent` macro and look for the `Make Array` node.
6. Drag off of the outputs of any `BuildDataString` macros and connect them to the inputs of the `Make Array` node.

### Example

In this example, a `PlayerSpawned` event is emitted when a player is spawned containing the ID of the player, stored as `PlayerID`. Any actors that are set up to listen for it will receive the event and have access to the ID of the player.

![Emitting an event](https://raw.githubusercontent.com/GyozaGuy/ark-observer/master/Examples/Emitting.png)

## Listening for an Event

To listen for an event, follow these steps ([click here for an example](https://github.com/GyozaGuy/ark-observer#listening-for-an-event-1)):
1. Get a reference to `GameMode` and cast it to `ShooterGameMode`.
2. Drag off of the cast to `ShooterGameMode` node and look for the `Bind Event to OnActorCustomEvent` node.
3. Drag off of the red method box on the event node and select the option to create a new custom event. Name it whatever you want.
4. Drag off of the `Event Custom String` output of the custom event node cast node and look for the `HandleEvent` macro.
5. Drag off of the `Event Name` output of the `HandleEvent` macro and use a `Switch on String` node to direct execution to different logic depending on the event name.
6. Connect the execution of the custom event node to the `Switch on String` node.
7. To get the event data corresponding to a specific key, use the `GetEventData` macro and connect its input to the `Event Data` output of the `HandleEvent` macro.
8. Enter the key for the data you want into the `Key` input of the `GetEventData` macro.
9. The `Value` output of the `GetEventData` macro will be the data you requested with the provided `Key` value, assuming it exists in the event data.

### Example

On the actor that is listening for the `PlayerSpawned` event, we get the `PlayerID` data from the event data and print it.

![Listening for an event](https://raw.githubusercontent.com/GyozaGuy/ark-observer/master/Examples/Listening.png)

# Things to be aware of

- I recommend emitting events and listening for events on the server to avoid duplication. Handle it however you want to though.
