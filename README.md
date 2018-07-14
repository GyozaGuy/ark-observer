# ARK Observer
A simple singleton to use in ARK modding projects that provides Observer Pattern-esque functionality. This is not a true Observer Pattern implementation, but uses many of the same concepts.

This provides a more robust way to pass data between actors in a mod. This even works between different mods, which is useful because the actors using it don't need to have references to anything they are passing the data to.

## Useful Reading

**Observer Pattern:** [https://en.wikipedia.org/wiki/Observer_pattern](https://en.wikipedia.org/wiki/Observer_pattern)

**Singleton Pattern:** [https://en.wikipedia.org/wiki/Singleton_pattern](https://en.wikipedia.org/wiki/Singleton_pattern)

# Setup Instructions

1. Download the `ObserverSingleton.uasset` file.
2. Import the downloaded file into your project.
3. Rename the imported file to avoid conflicts with other observers in other projects (if you have any).
4. Open your `PrimalGameData` file and add your renamed `ObserverSingleton` to your list of singleton actors.

# Usage Instructions

You can modify the `ObserverSingleton` directly if you wish, but I recommend leaving it alone. Instead, reference it in other graphs to gain access to its methods. This is especially important if you want to easily update it with future releases.

There are five methods on the `ObserverSingleton` to be aware of:
1. `EmitEvent` - emits an event that other actors can listen for. You need to specify an event name and any event data you wish to send.
2. `HandleEvent` - takes input from the `ActorCustomEvent` node (available in the ARK Dev Kit) and processes it, returning the event name and event data.
3. `GetData` - takes the event data (previously processed by `HandleEvent`) and a key and returns the event data that corresponds to that key.
4. `BuildEventString` - takes the event data and an event name and returns a string representing the event with its data. This happens automatically in `EmitEvent`, but I've made this method available in case you would like to get a raw event for some reason.
5. `BuildDataString` - takes a name and a value corresponding to the data you want to send and creates a data string using the name and value. If you don't want to use this method, you can easily create the data string yourself by putting the delimiter `:=:` between the name and the value (e.g. `PlayerName:=:MyPlayerName`). The `:=:` delimiter is to reduce the possibility of data containing characters that could conflict with the observer. **If you choose to create your data string manually, it is up to you to sanitize your data to make sure it doesn't include the following delimiters: `#$#`, `:=:`, `!@!` (used internally)**

## Emitting an Event

To emit an event, follow these steps ([click here for an example](https://github.com/GyozaGuy/ark-observer#emitting-an-event-1)):
1. Prepare the data you wish to send.
2. Get a reference to the `ObserverSingleton` actor, and cast it to the `ObserverSingleton` class.
3. Drag off of the `ObserverSingleton` cast node and find the `BuildDataString` method.
4. Enter the name of your data into the `Name` argument of the `BuildDataString` method. Enter the data itself into the `Value` argument.
5. Repeat steps 3 and 4 for all data you want to add to the event.
6. Drag off of the `Cast to ObserverSingleton` node, and look for the `EmitEvent` method.
7. Enter an event name directly into the `EmitEvent` node in the appropriate box.
8. Drag off of the `Data` argument on the `EmitEvent` node and look for the `Make Array` node.
9. Drag off of the output of any `BuildDataString` methods and connect them to the inputs of the `Make Array` node.

### Example

The `ObserverSingleton.uasset` file has a usage example built in if you need a reference while in the Dev Kit.

In this example, a player is given a buff, and we emit an event telling the universe the ID of the player (stored as `PlayerID`) that got the buff. Any actors that are set up to listen for it will receive the event and have access to the ID of the player.

![Emitting an event](https://raw.githubusercontent.com/GyozaGuy/ark-observer/master/Examples/Emitting.png)

## Listening for an Event

To listen for an event, follow these steps ([click here for an example](https://github.com/GyozaGuy/ark-observer#listening-for-an-event-1)):
1. Get a reference to `GameMode` and cast it to `ShooterGameMode`.
2. Drag off of the cast to `ShooterGameMode` node and look for the `Bind Event to OnActorCustomEvent` node.
3. Drag off of the red method box on the event node and select the option to create a new custom event. Name it whatever you want.
4. Get a reference to the `ObserverSingleton` actor, and cast it to the `ObserverSingleton` class.
5. Drag off of the cast node and look for the `HandleEvent` method.
6. Connect the `Event Custom String` input from the custom event node you created to the `Event String` argument of the `HandleEvent` method.
7. Drag off of the `Event Name` output of the `HandleEvent` method and use a `Switch on String` node to direct execution to different logic depending on the event name.
8. To get the event data corresponding to a specific key, drag off of the output of the `Cast to ObserverSingleton` node and find the `GetData` method.
9. Connect the `Event Data` output of the `HandleEvent` method to the `Event Data` input of the `GetData` method.
10. Enter the key for the data you want into the `Key` argument of the `GetData` method.
11. The `Value` output of the `GetData` method will be the data you requested with the provided `Key` value, assuming it exists in the event data.

### Example

On the actor that is listening for the event, we get the `PlayerID` data from the event data and print it.

![Listening for an event](https://raw.githubusercontent.com/GyozaGuy/ark-observer/master/Examples/Listening.png)

# Things to be aware of

- I recommend emitting events and listening for events on the server to avoid duplication. Handle it however you want to though.
- If you are going to listen for a lot of events, I suggest getting the reference to the `ObserverSingleton` in the `Event Begin Play` logic of your graph and storing it with a variable. Use that variable in the `OnActorCustomEvent` custom event you create to avoid getting the reference more times than you need to. (The same principle applies to emitting events... if you are going to use it a lot in the same graph, get the reference once and store it as a variable.)
- To get a reference to the `ObserverSingleton`, it _should_ be safe to use the `Get All Actors Of Class` method using the `ObserverSingleton` class, then getting the first element of the resulting array. It's a singleton, so only one should exist. Even if other mods use this singleton, it should be safe because in theory all potential `ObserverSingleton` actors in the game should have the exact same methods available for use.
- It is probably a good idea to use a small delay before emitting an event after your mod does something, especially in the case in the example where I'm spawning a buff and then listening for an event on that buff. The delay gives the buff a chance to be created.
