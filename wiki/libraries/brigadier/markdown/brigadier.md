# Brigadier

Licensing: The code in this article is licensed under the "Creative Commons Zero v1.0 Universal" license. The license grants you the rights to use the code examples shown in this article in your own mods.

====== Creating Commands ======

Creating commands can allow a mod developer to add functionality that can used through a command. 
This tutorial will teach you how to register commands, and the general command structure of Brigadier.

Note: All code written here was written for 1.14.4. Some mappings may have changed in yarn, but all code should still be applicable.

===== What is Brigadier? =====

Brigadier is a command parser & dispatcher written by Mojang for use in Minecraft. Brigadier is a tree based command library where you build a tree of arguments and commands.

The source code for brigadier can be found here: https://github.com/Mojang/brigadier

===== What is a command? =====

Brigadier requires you specify the ```Command``` to be run. A "command" is a fairly loose term within brigadier, but typically it means an exit point of the command tree. This is where the code is executed for your command.
A ''Command'' is a functional interface. The command has a generic type of ''S'' which defines the type of the command source. The command source provides some context in which a command was ran. In Minecraft, this is typically a ''ServerCommandSource'' which can represent a server, a command block, rcon connection, a player or an entity.

The single method in ```Command```, ```run(CommandContext<S>)``` takes a ```CommandContext<S>``` as the sole parameter and returns an integer. The command context holds your command source of ''S'' and allows you to obtain arguments, look at the parsed command nodes and see the input used in this command.

A command can be implemented in several ways as shown below:

**__As a lambda__**
```java
Command<Object> command = context -> {
    return 0;
};
```

**__As an anonymous class__**
```java
Command<Object> command = new Command<Object>() {
    @Override
    public int run(CommandContext<Object> context) {
        return 0;
    }
}
```

**__Implemented as a class__**
```java
final class XYZCommand implements Command<Object> {
    @Override
    public int run(CommandContext<Object> context) {
        return 0;
    }
}
```

**__As a method reference__**
```java
void registerCommand() {
    // Ignore this for now, we will explain it next.
    dispatcher.register(CommandManager.literal("foo")
        .executes(this::execute)); // This refers to the "execute" method below.
}

private int execute(CommandContext<Object> context) {
    return 0;
}
```


The ```run(CommandContext)``` method can throw a ```CommandSyntaxException```, but this is covered later in the tutorial. 

The integer can be considered the result of the command. In Minecraft, the result can correspond to the power of a redstone comparator feeding from a command block or the value that will be passed the chain command block the command block is facing. Typically negative values mean a command has failed and will do nothing. A result of ''0'' means the command has passed. Positive values mean the command was successful and did something.

===== A basic command =====

Below is a command that contains no arguments:
```java
dispatcher.register(CommandManager.literal("foo").executes(context -> { 
    System.out.println("Called foo with no arguments");

    return 1;
}));
```

```CommandManager.literal("foo")``` tells brigadier this command has one node, a **literal** called ```foo```.
To execute this command, one must type ```/foo```. If ```/Foo```, ```/FoO```, ```/FOO```, ```/fOO``` or ```/fooo``` is typed instead, the command will not run.

===== Static Imports =====
Typing out ```CommandManager.literal("foo")``` every time you want to create a literal may feel redundant. You can clean up your codebase by use of static imports for the arguments. For a literal this would shorten the statement to ''literal("foo")''. This also works for getting the value of an argument. This shortens ```StringArgumentType.getString(ctx, "string")``` to ```getString(ctx, "string")```. This also works for Minecraft's own argument types.

Below is an example of some static imports:
```java
// getString(ctx, "string")
import static com.mojang.brigadier.arguments.StringArgumentType.getString;
// word()
import static com.mojang.brigadier.arguments.StringArgumentType.word;
 // literal("foo")
import static net.minecraft.server.command.CommandManager.literal;
 // argument("bar", word())
import static net.minecraft.server.command.CommandManager.argument;
// Import everything
import static net.minecraft.server.command.CommandManager.*;
```

Note: Please be sure you use the ```literal``` and ```argument``` from ```CommandManager``` or you may have issues with generics when trying to compile.

Brigadier's default arguments are at ```com.mojang.brigadier.arguments```

Minecraft's arguments are in ```net.minecraft.command.arguments```.
CommandManager is in the package ```net.minecraft.server.command```

==== A sub command ====

To add a sub command, you register the first literal node of the command normally.

```java
dispatcher.register(CommandManager.literal("foo")
```
In order to have a sub command, one needs to append the next node to the existing node. This is done use the ```then(ArgumentBuilder)``` method which takes in an ```ArgumentBuilder''.

This creates the command ''foo <bar>'' as shown below.

<code java [enable_line_numbers="true", highlight_lines_extra="2"]>
dispatcher.register(literal("foo")
    .then(literal("bar"))
);
</code>

It is advised to indent your code as you add nodes to the command. Usually the indentation corresponds to how many nodes deep one is on the command tree. The new line also makes it visible that another node is being added. There are alternative styles to formatting the tree command that are shown later on in this tutorial.

**So let's try running the command**

Most likely if you typed ''/foo bar'' in game, the command will fail to run. This is because there is no code for the game to execute when all the required arguments have been met. To fix this, you need to tell the game what to run when the command is being executed using the ''executes(Command)'' method. Below is how the command should look as an example.

<code java [enable_line_numbers="true", highlight_lines_extra="3,4,5,6,7"]>
dispatcher.register(literal("foo")
    .then(literal("bar")
        .executes(context -> {
            System.out.println("Called foo with bar");

            return 1;
        })
    )
);
</code>

===== Registering the commands =====

Registering commands is done by registering a callback using the ''CommandRegistrationCallback''. For information on registering callbacks, please see the [[tutorial:callbacks|callbacks article]].

The event should be registered in your mod's initializer. The callback has two parameters. The ''CommmandDispatcher<S>'' is used to register, parse and execute commands. ''S'' is the type of command source the command dispatcher supports. The second parameter is a boolean which identifies the type of server the commands are being registered. on is an ''dedicated'' or ''integrated'' (false) server.


<code java [enable_line_numbers="true"]>
public class ExampleCommandMod implements ModInitializer {
    @Override
    public void onInitialize() {
        CommandRegistrationCallback.EVENT.register((dispatcher, dedicated) -> {
            ...
        });
    }
}
</code>

Inside your lambda, method reference or whatever you have chosen, you will register your commands.

<code java [enable_line_numbers="true", highlight_lines_extra="5,6,7,8"]>
public class ExampleCommandMod implements ModInitializer {
    @Override
    public void onInitialize() {
        CommandRegistrationCallback.EVENT.register((dispatcher, dedicated) -> {
            dispatcher.register(CommandManager.literal("foo").executes(context -> {
                System.out.println("foo");
                return 1;
            }));
        });
    }
}
</code>

If desired, you can make sure a command is only registered on a dedicated server by checking the ''dedicated'' flag


<code java [enable_line_numbers="true", highlight_lines_extra="5,6,7"]>
public class ExampleCommandMod implements ModInitializer {
    @Override
    public void onInitialize() {
        CommandRegistrationCallback.EVENT.register((dispatcher, dedicated) -> {
            if (dedicated) {
                TestDedicatedCommand.register(dispatcher);
            }
        });
    }
}
</code>

And vice versa

<code java [enable_line_numbers="true", highlight_lines_extra="5,6,7"]>
public class ExampleCommandMod implements ModInitializer {
    @Override
    public void onInitialize() {
        CommandRegistrationCallback.EVENT.register((dispatcher, dedicated) -> {
            if (!dedicated) {
                TestIntegratedCommand.register(dispatcher);
            }
        });
    }
}
</code>

===== Arguments =====

Arguments in Brigadier both parse and error check any inputted arguments.
Minecraft creates some special argument types for it's own use such as the ''EntityArgumentType'' which represents the in-game entity selectors ''@a, @r, @p, @e[type=!player, limit=1, distance=..2]'', or an ''NbtTagArgumentType'' that parses stringified nbt (snbt) and verifies that the input is the correct syntax.

**TODO:** Go into more detail on how to use arguments

====== Advanced concepts ======

Below are links to the articles about more complex concepts used in brigadier.

^ Page                                                           ^ Description                                                                     ^
| [[tutorial:command_requirements|Requirements]]               | Only allow users to execute commands in certain scenarios.                      |
| [[tutorial:command_exceptions  |Exceptions]]                 | Fail execution of a command with a descriptive message and in certain contexts. |                                                                  
| [[tutorial:command_suggestions|Suggestions]]                | Suggesting command input for the client.                                        |
| [[tutorial:command_redirects_aliases|Redirects (Aliases)]]   | Allow use of aliases to execute commands.                                       |
| [[tutorial:command_redirects_chaining|Redirects (Chaining)]] | Allow commands to have repeating elements and flags.                            |
| [[tutorial:command_argument_types|Custom Argument Types]]    | Parse your own arguments into your own objects.                                 |

**TODO:** Sections are being moved to sub categories and will be added to their respective articles as they are migrated.

====== FAQ ======

===== Why does my command not compile =====

There are two immediate possibilities for why this could occur.

==== Catch or throw a CommandSyntaxException ====

The solution to this issue is to make the ''run'' or ''suggest'' methods throw a ''CommandSyntaxException''. Brigadier will handle the checked exceptions and forward the proper error message in game for you.

==== Issues with generics ====

You may have an issue with generic types once in a while. Verify you are using ''CommandManager.literal(...)'' or ''CommandManager.argument(...)'' instead ''LiteralArgumentBuilder'' or ''RequiredArgumentBuilder'' in your static imports.

===== Can I register client side commands? =====

Fabric has a ClientCommandManager that can be used to register client side commands.

===== Dark Arts =====

A few things we don't recommend, but are possible.

==== Can I register commands in runtime? ====

You can do this but it is not recommended. You would get the ''CommandManager'' from the server and add anything commands you wish to it's ''CommandDispatcher''.

After that you need to send the command tree to every player again using ''CommandManager.sendCommandTree(ServerPlayerEntity)''. This is required because the client locally caches the command tree it receives during login (or when operator packets are sent) for local completions rich error messages.

==== Can I unregister commands in runtime? ====

You can also do this, however it is much less stable than registering commands and could cause unwanted side effects. To keep things simple, you need to use reflection on brigadier and remove the nodes. After this, you need to send the command tree to every player again using ''sendCommandTree(ServerPlayerEntity)''. If you don't send the updated command tree, the client may think a command still exists, even though the server will fail execution.

----

====== Sorry for the mess ======

**__Currently this article is being migrated, so things may be a mess. Below is are the parts of the article that are yet to be migrated to the new format.__**

Licensing from below onwards is available under the "CC Attribution-Noncommercial-Share Alike 4.0 International" license. This is the current license of the other wiki articles.

===== Requirements =====

Lets say you have a command you only want operators to be able to execute. This is where the ''requires'' method comes into play. The requires method has one argument of a Predicate<ServerCommandSource> which will supply a ServerCommandSource to test with and determine if the CommandSource can execute the command.

For example this may look like the following:

<code java [enable_line_numbers="true"]>
dispatcher.register(literal("foo")
	.requires(source -> source.hasPermissionLevel(4))
		.executes(ctx -> {
			ctx.getSource().sendFeedback(Text.literal("You are an operator"), false);
			return 1;
		});
</code>

This command will only execute if the Source of the command is a level 4 operator at minimum. If the predicate returns false, then the command will not execute. Also this has the side effect of not showing this command in tab completion to anyone who is not a level 4 operator.

Nothing prevents someone from specifying calls to permissions implementations within the ''requires'' block. Just note that if permissions change, you need to re send the command tree.

===== Exceptions =====

Brigadier supports command exceptions which can be used to end a command such as if an argument didn't parse properly or the command failed to execute, as well as including richer details of the failure.

All the exceptions from Brigadier are based on the CommandSyntaxException. The two main types of exceptions Brigadier provides are Dynamic and Simple exception types, of which you must ''create()'' the exception to throw it. These exceptions also allow you to specify the context in which the exception was thrown using ''createWithContext(ImmutableStringReader)'', which builds the error message to point to where on the inputted command line the error occured.
Below is a coin flip command to show an example of exceptions in use.

<code java [enable_line_numbers="true"]>
dispatcher.register(CommandManager.literal("coinflip")
    .executes(ctx -> {
        Random random = new Random();
			
        if(random.nextBoolean()) { // If heads succeed.
            ctx.getSource().sendMessage(new TranslatableText("coin.flip.heads"))
            return Command.SINGLE_SUCCESS;
        }

        throw new SimpleCommandExceptionType(new TranslatableText("coin.flip.tails")).create(); // Oh no tails, you lose.
    }));
</code>

Though you are not just limited to a single type of exception as Brigadier also supplies Dynamic exceptions which take additional parameters for context.

<code java [enable_line_numbers="true"]>
DynamicCommandExceptionType used_name = new DynamicCommandExceptionType(name -> {
    return Text.literal("The name: " + (String) name + " has been used");
});
</code>

There are more Dynamic exception types which each take a different amount of arguments into account (''Dynamic2CommandExceptionType'', ''Dynamic3CommandExceptionType'', ''Dynamic4CommandExceptionType'', ''DynamicNCommandExceptionType'').
You should remember that the Dynamic exceptions takes an object as an argument so you may have to cast the argument for your use.

===== Redirects (Aliases) =====

Redirects are Brigadier's form of aliases. Below is how Minecraft handles /msg have an alias of /tell and /w. 

<code java [enable_line_numbers="true"]>
public static void register(CommandDispatcher<ServerCommandSource> dispatcher) {
    LiteralCommandNode node = registerMain(dispatcher); // Registers main command
    dispatcher.register(literal("tell")
        .redirect(node)); // Alias 1, redirect to main command
    dispatcher.register(literal("w")
        .redirect(node)); // Alias 2, redirect to main command
}

public static LiteralCommandNode registerMain(CommandDispatcher<ServerCommandSource> dispatcher) {
    return dispatcher.register(literal("msg")
    .then(argument("targets", EntityArgumentType.players())
        .then(argument("message", MessageArgumentType.message())
            .executes(ctx -> {
                return execute(ctx.getSource(), getPlayers(ctx, "targets"), getMessage(ctx, "message"));
            }))));
}
</code>

The redirect tells brigadier to continue parsing the command at another command node.

===== Redirects (Chainable Commands) =====
Commands such as ''/execute as @e[type=player] in the_end run tp ~ ~ ~'' are possible because of redirects. Below is an example of a chainable command:

<code java [enable_line_numbers="true"]>
LiteralCommandNode<ServerCommandSource> root = dispatcher.register(literal("fabric_test"));
LiteralCommandNode<ServerCommandSource> root1 = dispatcher.register(literal("fabric_test") 
// You can register under the same literal more than once, it will just register new parts of the branch as shown below if you register a duplicate branch an error will popup in console warning of conflicting commands but one will still work.
    .then(literal("extra")
        .then(literal("long")
            .redirect(root, this::lengthen)) // Return to root for chaining
        .then(literal("short")
            .redirect(root, this::shorten))) // Return to root for chaining
        .then(literal("command")
            .executes(ctx -> {
                ctx.getSource().sendFeedback(Text.literal("Chainable Command"), false);
                return Command.SINGLE_SUCCESS;
})));
</code>
The redirect can also modify the CommandSource by use of a ''redirect modifier'' which can be used for builder commands.

<code java [enable_line_numbers="true"]>
.redirect(rootNode, context -> {
    return ((ServerCommandSource) context.getSource()).withLookingAt(Vec3ArgumentType.getVec3(context, "pos"));
})
</code>

===== What can the ServerCommandSource do? =====

A server command source provides some additional implementation specific context when a command is run. This includes the ability to get the entity that executed the command, the world the command was ran in or the server the command was run on.

<code java [enable_line_numbers="true"]>
final ServerCommandSource source = ctx.getSource(); 
// Get the source. This will always work.

final Entity sender = source.getEntity(); 
// Unchecked, may be null if the sender was the console.

final Entity sender2 = source.getEntityOrThrow(); 
// Will end the command if the source of the command was not an Entity. 
// The result of this could contain a player. Also will send feedback telling the sender of the command that they must be an entity. 
// This method will require your methods to throw a CommandSyntaxException. 
// The entity options in ServerCommandSource could return a CommandBlock entity, any living entity or a player.

final ServerPlayerEntity player = source.getPlayer(); 
// Will end the command if the source of the command was not explicitly a Player. Also will send feedback telling the sender of the command that they must be a player.  This method will require your methods to throw a CommandSyntaxException

source.getPosition(); 
// Get's the sender's position as a Vec3 when the command was sent. This could be the location of the entity/command block or in the case of the console, the world's spawn point.

source.getWorld(); 
// Get's the world the sender is within. The console's world is the same as the default spawn world.

source.getRotation(); 
// Get's the sender's rotation as a Vec2f.

source.getMinecraftServer(); 
// Access to the instance of the MinecraftServer this command was ran on.

source.getName(); 
// The name of the command source. This could be the name of the entity, player, the name of a CommandBlock that has been renamed before being placed down or in the case of the Console, "Console"

source.hasPermissionLevel(int level); 
// Returns true if the source of the command has a certain permission level. This is based on the operator status of the sender. (On an integrated server, the player must have cheats enabled to execute these commands)
</code>

===== Some example commands examples =====

=== Broadcast a message ===

<code java [enable_line_numbers="true"]> 
public static void register(CommandDispatcher<ServerCommandSource> dispatcher){
    dispatcher.register(literal("broadcast")
        .requires(source -> source.hasPermissionLevel(2)) // Must be a game master to use the command. Command will not show up in tab completion or execute to non operators or any operator that is permission level 1.
            .then(argument("color", ColorArgumentType.color())
                .then(argument("message", greedyString())
                    .executes(ctx -> broadcast(ctx.getSource(), getColor(ctx, "color"), getString(ctx, "message")))))); // You can deal with the arguments out here and pipe them into the command.
}

public static int broadcast(ServerCommandSource source, Formatting formatting, String message) {
    final Text text = Text.literal(message).formatted(formatting);

    source.getMinecraftServer().getPlayerManager().broadcastChatMessage(text, MessageType.CHAT, source.getPlayer().getUuid());
    return Command.SINGLE_SUCCESS; // Success
}
</code>

==== /giveMeDiamond ====

First the basic code where we register "giveMeDiamond" as a literal and then an executes block to tell the dispatcher which method to run.

<code java [enable_line_numbers="true"]>
public static LiteralCommandNode register(CommandDispatcher<ServerCommandSource> dispatcher) { // You can also return a LiteralCommandNode for use with possible redirects
    return dispatcher.register(literal("giveMeDiamond")
        .executes(ctx -> giveDiamond(ctx)));
}
</code>

Then since we only want to give to players, we check if the CommandSource is a player. But we can use ''getPlayer'' and do both at the same time and throw an error if the source is not a player.

<code java [enable_line_numbers="true"]>
public static int giveDiamond(CommandContext<ServerCommandSource> ctx) throws CommandSyntaxException {
    final ServerCommandSource source = ctx.getSource();
		
    final PlayerEntity self = source.getPlayer(); // If not a player than the command ends
</code>

Then we add to the player's inventory, with a check to see if the inventory is full:

<code java [enable_line_numbers="true"]>
    if(!player.inventory.insertStack(new ItemStack(Items.DIAMOND))){
        throw new SimpleCommandExceptionType(new TranslatableText("inventory.isfull")).create();
    }

    return 1;
}
</code>

==== Antioch ====
...lobbest thou thy Holy Hand Grenade of Antioch towards thy foe.
who being naughty in My sight, shall snuff it.

Aside from the joke this command summons a primed TNT to a specified location or the location of the sender's cursor.

First create an entry into the CommandDispatcher that takes a literal of antioch with an optional argument of the location to summon the entity at.

<code java [enable_line_numbers="true"]>
public static void register(CommandDispatcher<ServerCommandSource> dispatcher) {
    dispatcher.register(literal("antioch")
        .then(required("location", BlockPosArgumentType.blockPos()
            .executes(ctx -> antioch(ctx.getSource(), BlockPosArgument.getBlockPos(ctx, "location")))))
        .executes(ctx -> antioch(ctx.getSource(), null)));
}
</code>

Then the creation and messages behind the joke.

<code java [enable_line_numbers="true"]>
public static int antioch(ServerCommandSource source, BlockPos blockPos) throws CommandSyntaxException {
    if(blockPos == null) {
        // For the case of no inputted argument we calculate the cursor position of the player or throw an error if the nearest position is too far or is outside of the world.
        // This class is used as an example and actually doesn't exist yet.
        blockPos = LocationUtil.calculateCursorOrThrow(source, source.getRotation());
    }

    final TntEntity tnt = new TntEntity(source.getWorld(), blockPos.getX(), blockPos.getY(), blockPos.getZ(), null);
    tnt.setFuse(3);
        
    source.getServer().getPlayerManager().broadcastChatMessage(Text.literal("...lobbest thou thy Holy Hand Grenade of Antioch towards thy foe", MessageType.CHAT, UUID.randomUUID()).formatted(Formatting.RED), false);
    source.getServer().getPlayerManager().broadcastChatMessage(Text.literal("who being naughty in My sight, shall snuff it.", MessageType.CHAT, UUID.randomUUID()).formatted(Formatting.RED), false);
    source.getWorld().spawnEntity(tnt);
    return 1;
}
</code>

==== More examples coming soon ====

===== Custom Argument Types =====

Brigadier has support for custom argument types and this section goes into showing how to create a simple argument type. 

Warning: Custom arguments require client mod installation to be registered correctly! If you are making a server plugin, consider using existing argument type and a custom suggestions provider instead.

For this example we will create a UuidArgumentType.

First create a class which extends ''ArgumentType''. Note that ArgumentType is a generic, so the generic will define what type the ArgumentType will return

<code java [enable_line_numbers="true"]>
public class UuidArgumentType implements ArgumentType<UUID> {
</code>

ArgumentType requires you to implement the ''parse'' method, the type it returns will match with the Generic type.
<code java>
@Override
public UUID parse(StringReader reader) throws CommandSyntaxException {
</code>

This method is where all of your parsing will occur. Either this method will return the object based on the arguments provided in the command line or throw a CommandSyntaxException and parsing will fail.

Next you will store the current position of the cursor, this is so you can substring out only the specific argument. This will always be at the beginning of where your argument appears on the command line.

<code java [enable_line_numbers="true"]>
int argBeginning = reader.getCursor(); // The starting position of the cursor is at the beginning of the argument.
if (!reader.canRead()) {
    reader.skip();
}
</code>

Now we grab the entire argument. Depending on your argument type, you may have a different criteria or be similar to some arguments where detecting a ''{'' on the command line will require it to be closed. For a UUID we will just figure out what cursor position the argument ends at.

<code java [enable_line_numbers="true"]>
while (reader.canRead() && reader.peek() != ' ') { // peek provides the character at the current cursor position.
    reader.skip(); // Tells the StringReader to move it's cursor to the next position.
}
</code>

Then we will ask the StringReader what the current position of the cursor is an substring our argument out of the command line.

<code java [enable_line_numbers="true"]>String uuidString = reader.getString().substring(argBeginning, reader.getCursor());</code>

Now finally we check if our argument is correct and parse the specific argument to our liking, and throwing an exception if the parsing fails.

<code java [enable_line_numbers="true"]>
try {
    UUID uuid = UUID.fromString(uuidString); // Now our actual logic.
    return uuid; // And we return our type, in this case the parser will consider this argument to have parsed properly and then move on.
    } catch (Exception ex) {
    // UUIDs can throw an exception when made by a string, so we catch the exception and repackage it into a CommandSyntaxException type.
    // Create with context tells Brigadier to supply some context to tell the user where the command failed at.
    // Though normal create method could be used.
    throw new SimpleCommandExceptionType(Text.literal(ex.getMessage())).createWithContext(reader);
}
</code>

The ArgumentType is done, however your client will refuse the parse the argument and throw an error. This is because the server will tell the client what argument type the command node is. And the client will not parse any argument types it does not know how to parse. To fix this we need to register an ArgumentSerializer. 
Within your ModInitializer. For more complex argument types, you may need to create your own ArgumentSerializer.

<code java [enable_line_numbers="true"]>
ArgumentTypeRegistry.register(new Identifier("mymod", "uuid"), UuidArgumentType.class, ConstantArgumentSerializer.of(UuidArgumentType::uuid)); 
// The argument should be what will create the ArgumentType.
</code>

And here is the whole ArgumentType:

<file java UuidArgumentType.java [enable_line_numbers="true"]>

import com.mojang.brigadier.StringReader;
import com.mojang.brigadier.arguments.ArgumentType;
import com.mojang.brigadier.context.CommandContext;
import com.mojang.brigadier.exceptions.CommandSyntaxException;
import com.mojang.brigadier.exceptions.SimpleCommandExceptionType;
import net.minecraft.text.Text;
import net.minecraft.util.SystemUtil;

import java.util.ArrayList;
import java.util.Collection;
import java.util.UUID;

/**
 * Represents an ArgumentType that will return a UUID.
 */
public class UuidArgumentType implements ArgumentType<UUID> {
    public static UuidArgumentType uuid() {
        return new UuidArgumentType();
    }

    public static <S> UUID getUuid(String name, CommandContext<S> context) {
        // Note that you should assume the CommandSource wrapped inside of the CommandContext will always be a generic type.
        // If you need to access the ServerCommandSource make sure you verify the source is a server command source before casting.
        return context.getArgument(name, UUID.class);
    }

    private static final Collection<String> EXAMPLES = SystemUtil.consume(new ArrayList<>(), list -> {
        list.add("765e5d33-c991-454f-8775-b6a7a394c097"); // i509VCB: Username The_1_gamers
        list.add("069a79f4-44e9-4726-a5be-fca90e38aaf5"); // Notch
        list.add("61699b2e-d327-4a01-9f1e-0ea8c3f06bc6"); // Dinnerbone
    });

    @Override
    public UUID parse(StringReader reader) throws CommandSyntaxException {
        int argBeginning = reader.getCursor(); // The starting position of the cursor is at the beginning of the argument.
        if (!reader.canRead()) {
            reader.skip();
        }

        // Now we check the contents of the argument till either we hit the end of the command line (When canRead becomes false)
        // Otherwise we go till reach reach a space, which signifies the next argument
        while (reader.canRead() && reader.peek() != ' ') { // peek provides the character at the current cursor position.
            reader.skip(); // Tells the StringReader to move it's cursor to the next position.
        }

        // Now we substring the specific part we want to see using the starting cursor position and the ends where the next argument starts.
        String uuidString = reader.getString().substring(argBeginning, reader.getCursor());
        try {
            UUID uuid = UUID.fromString(uuidString); // Now our actual logic.
            return uuid; // And we return our type, in this case the parser will consider this argument to have parsed properly and then move on.
        } catch (Exception ex) {
            // UUIDs can throw an exception when made by a string, so we catch the exception and repackage it into a CommandSyntaxException type.
            // Create with context tells Brigadier to supply some context to tell the user where the command failed at.
            // Though normal create method could be used.
            throw new SimpleCommandExceptionType(Text.literal(ex.getMessage())).createWithContext(reader);
        }
    }

    @Override
    public Collection<String> getExamples() { // Brigadier has support to show examples for what the argument should look like, this should contain a Collection of only the argument this type will return. This is mainly used to calculate ambiguous commands which share the exact same 
        return EXAMPLES;
    }
}
</file>
