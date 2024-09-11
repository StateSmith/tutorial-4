# Let's Create A Brand New State Machine
In this lesson, we will create a new state machine from scratch that controls a laser flashlight.

You can see it in action by opening `index.html` in a web browser.

![](./docs/intro.png)

## Delete Existing `LightSm.*` Files
Our goal is to create a new state machine from scratch so go ahead and delete the existing `LightSm.*` files in this directory.


## Create a New `.drawio` File
We can do this manually, or `ss.cli` can help us by generating a new diagram from template for us.

Let's use `ss.cli`. Run the following command in the terminal:

```bash
ss.cli create
```


Select `User Friendly` and hit enter.<br>
![](docs/select-user-friendly.png)

Type in the name of the state machine `LightSm` and hit enter.<br>
📢 NOTE! It must be exactly `LightSm` for the rest of this lesson to work.<br>
![](docs/type-light-sm.png)

Select `JavaScript` and hit enter.<br>
![](docs/select-js.png)

Select `.drawio` and hit enter.<br>
![](docs/select-drawio.png)

Take the default suggestion for diagram name/path by hitting enter.<br>
![](docs/take-diagram-path-suggestion.png)

Select the `Multiple pages` template and hit enter.<br>
![](docs/select-template.png)

Select `yes` to confirm and hit enter.<br>
![](docs/confirm.png)

You should now see a `LightSm.drawio` file in this directory.


<br>




## Review draw.io Usage Tips 📝
Review the [animated gif instructions here](https://github.com/StateSmith/StateSmith/wiki/draw.io:-tips) for the basics of using draw.io.

Stuff like how to:
- add states
- add transitions
- modify labels
- ...

![](docs/usage-tips.png)

### 📢 Important! Avoid Common Mistake
The tips link above has an important note about [avoiding the common "Non-related nodes overlap" mistake](https://github.com/StateSmith/StateSmith/wiki/Troubleshooting#non-related-nodes-overlap) when using draw.io.

This is probably the most common issue new users bump into when using draw.io to create state machines. Easy to avoid once you know about it.


<br>


## Edit the Diagram File

Modify the design page to look like this:

![](docs/1.png)

We now have a state machine with two states `OFF` and `ON1`. The `OFF` state is the initial state and the `ON1` state is the state when the laser flashlight is on.

Replace the config page TOML settings with this:
    
```toml
SmRunnerSettings.transpilerId = "JavaScript"
```

![](docs/1-config.png)


<br>


# Generate The State Machine Code
Run the below command in this directory to generate code from your diagram file.
```
ss.cli run --here
```


<br>


## Interact With The State Machine 🌍
Now that we've generated the state machine JavaScript code, let's interact with it.
Open `index.html` with a web browser. You may need to refresh the page in your browser to see the changes.

You should now be able to use the `INC` and `DIM` buttons to control the laser flashlight.


<br>
<br>


# Add Another State `ON2`
Add another state `ON2` to the design with the following behavior:
* upon entering the state, it should call the function `yellowLaser()`
* it should transition to `ON1` when the `DIM` event is received

Additionally, add a transition from `ON1` to `ON2` when the `INC` event is received.

See if you can do it yourself. If you get stuck, you can find the solution below.

Remember the [usage tips link](https://github.com/StateSmith/StateSmith/wiki/draw.io:-tips).

 *scroll down* ▼ <br>
 <br>
 <br>
 <br>
 <br>
 *scroll down* ▼ <br>
 <br>
 <br>
 <br>
 <br>
 *scroll down* ▼ <br>
 <br>
 <br>
 <br>
 <br>
 *scroll down* ▼ <br>
 <br>
 <br>
 <br>
 <br>
 
![](docs/2.png)



Generate your state machine code using `ss.cli` and open `index.html` in a web browser to interact with the state machine. You may need to refresh the page in your browser to see the changes.



<br>
<br>

# Add A Variable And Guard Condition
We have one more laser color to add to our flashlight: `redLaser()`.

To get to the red laser `ON3` state, we want the user to have to press the `INC` button three times while in the `ON2` state.

There's a number of ways to implement this. We could use a global count variable, but we will use a variable that belongs to the state machine instance instead.

### Add The `ON3` State
Add the `ON3` state to the design like all the others. We will modify it later to use the `count` variable.

![](docs/3.png)


### Add A `count` Variable
Add this bit of TOML to your `$CONFIG : toml` section in the `config` page:

```toml
RenderConfig.AutoExpandedVars = """
    count: 0,
    """
```

![](docs/declare-count-var.png)

Note the following about the `AutoExpandedVars` section:
* We are using a TOML multi-line string here so that it is easy to add more than one variable.
* The variable is in the `AutoExpandedVars` section so we can reference it in the diagram naturally instead of writing `this.vars.count` or something similar.
* 📢 **<u>This</u> declaration syntax is programming language specific**. If we were generating C/C++/C#/Java/Python... (instead of JS), we would use different syntax to initialize the variable. Use the syntax appropriate to your target programming language.
More info [here](https://github.com/StateSmith/StateSmith/blob/main/docs/settings.md#renderconfigautoexpandedvars).


### Reset The `count` Variable Upon Entering `ON2`
Add the following behavior to the `ON2` state:

```
enter / count = 0;
```

This will reset the `count` variable to `0` every time we enter the `ON2` state.

![](docs/zero-count.png)

### Increment The `count` Variable
Add the following behavior to the `ON2` state handler:

```
INC / count++;
```

This will increment the `count` variable every time the `INC` event is received while in the `ON2` state.

![](docs/inc-count.png)

### Transition To `ON3` When `count` Is `3`
Modify the transition from `ON2` to `ON3` to include a guard condition `[count >= 3]`.

![](docs/add-guard.png)


### Ensure Event Handling Order
We now have multiple `ON2` behaviors that are triggered by the `INC` event. Sometimes we don't care about their execution order, but in this specific case, we need to ensure that the `count++` behavior is executed before the transition behavior guard `[count >= 3]` is evaluated. Otherwise, it may take four `INC` events to transition to `ON3`.

We can do this with a small StateSmith ordering extension to UML syntax. Simply add a `1.` before the `INC` event:

```
1. INC / count++;
```

We don't need to order the other transition behavior because unordered behaviors are executed after ordered behaviors.

![](docs/add-order.png)

If you accidentally put the `1.` in the wrong place, you will either get a StateSmith error message, or your generated code will have a syntax error:

![](./docs/order-error-ex.png)

If you are ever unsure about the syntax, you can always refer to the StateSmith [behavior documentation](https://github.com/StateSmith/StateSmith/wiki/Behaviors).

### Check Your Work
You design should now look like this:

![](docs/check-after-3.png)




### Generate The Code
Generate the code using `ss.cli` and open `index.html` in a web browser to interact with the state machine. You may need to refresh the page in your browser to see the changes.


<br>
<br>

# Polling A Timeout Transition
Let's assume that the `ON3` state will cause the flashlight to overheat if it stays in that state for too long. We will add a timeout transition that will transition to the `OFF` state if the flashlight stays in the `ON3` state for more than 5 seconds.

We could implement this many ways, but I want to show how you can use "polling" to check guard conditions.

StateSmith state machines are purely event driven. They just sit waiting for you to feed them the next event (no thread, or background CPU usage).

If you need to poll certain conditions, simply send the `DO` event to your state machine at the rate you want. You have full control over how and when the state machine runs.

Any transition without an event specified will use the `DO` event. This is different than UML, but we found the UML way caused problems for most users.



### Add Transition To `OFF` State
A future section will cover how to use the current time to implement a timeout transition. For now, we will simply re-use the `count` variable to implement a timeout transition.

The `index.html` JavaScript is already setup to dispatch the `DO` event to the state machine every 500ms. All we need to do is wait for 10 `DO` events to transition to the `OFF` state. Why 10? 5 seconds x 2 events/second.

Add the following behaviors to the `ON3` state:

- `enter / count = 5 * 2;`
- `1. do / count--;`
- Transition from `ON3` to `OFF` when `count <= 0`

It should look something like this:

![](docs/on3-to-off.png)

We see that the transition from `ON3` to `OFF` is guarded by `[count <= 0]`, but there is no event explicitly associated with it like `DIM` or `INC`.

For convenience, StateSmith will *implicitly* assume the `DO` event for any behavior/transition that does not have an explicit event associated with it.

You can see this if you open the `LightSm.sim.html` file in your browser and look at the `ON3` state (after you run code generation again). You will see that the transition to `OFF` is guarded by `DO [count <= 0]`.

### Polled State Machines
In case the last section wasn't clear, polled state machines need the `DO` event to be dispatched to them periodically. This enables them to check guard conditions.

If, however, you want a behavior/transition to be checked for any event/trigger, check out StateSmith [TriggerMaps](https://github.com/StateSmith/StateSmith/issues/161).


### Test It
Generate the code using `ss.cli` and open `index.html` in a web browser to interact with the state machine. You may need to refresh the page in your browser to see the changes.


<br>
<br>

# Add A Composite State
Let's finally wire up the `OFF` event.

Instead of having each of the 3 `ON` states transition to `OFF`, we will create a composite state `ON_GROUP` that will contain the `ON1`, `ON2`, and `ON3` states. The `ON_GROUP` state will transition to the `OFF` state when the `OFF` event is received.

### Add The `ON_GROUP` State
Add the `ON_GROUP` state and move the `ON1`, `ON2`, and `ON3` states into it.

> TIP: if you haven't already watched the short tip on [Non-related nodes overlap](https://github.com/StateSmith/StateSmith/wiki/Troubleshooting#non-related-nodes-overlap), you probably should.

The [StateSmith draw.io plugin makes grouping](https://github.com/StateSmith/StateSmith-drawio-plugin/wiki/How-to-Use#group-states) states much faster, but we will do it manually here. We still don't have a great way of [installing](https://github.com/StateSmith/StateSmith/issues/368) the StateSmith plugin into draw.io.

It's not hard to add a new state manually. It just takes a few more steps.
1. off to the side, create a new state that will be the group.
1. drag the states you want to group into the new state.
1. move the group to where you want it.
1. some edges might be "behind" the new group state, so you might need to move them to the front.

<br>

![group](https://github.com/user-attachments/assets/483e9473-fa85-4f58-9405-a6caa3186c3b)

> Tip from above gif: you can reset the edge type to `Orthogonal` to have it auto route the edge if it needs adjusting.
>
> ![image](https://github.com/user-attachments/assets/0c44876f-9bb9-4bca-8710-afafe9714ddc)


### Add The Transition
Add the transition from `ON_GROUP` to `OFF` when the `OFF` event is received.

![add-off-transition](https://github.com/user-attachments/assets/628c19f8-d67a-4ee6-8b60-a8436379c63e)

> Tip from above gif: sometimes the `Straight` edge type looks better.


And that's it! Normally handwriting a hierarchical state machine is a lot more work. Easy for us.

### Test It
Generate the code using `ss.cli` and open `index.html` in a web browser to interact with the state machine. You may need to refresh the page in your browser to see the changes.

<br>

## Add Some Style
If you want, you can add style to highlight specific states and transitions.

![](docs/styled.png)

<br>

# Onwards! ⏭️
Congratulations! You've created a brand new state machine from scratch that controls a laser flashlight.

In the next section, we will discuss options on how to connect your state machine to the outside world.

Head on over to [lesson 3 README.md](../lesson-3/README.md).
