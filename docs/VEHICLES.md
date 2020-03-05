# Modding with vehicles

In this tutorial, you're going to learn how to add ManureSystem support to your vehicle mod.

### What do I need?

To successfully execute the required steps in this tutorial you will need the following programs: 

- Text editor ([Notepad++](https://notepad-plus-plus.org/downloads/), [Visual Studio Code](https://code.visualstudio.com/) or any proper IDE ...)
- GIANTS Editor 8.2.0 or 3D software that supports the GIANTS Exporter (Maya, blender ...)
- Internet connection for downloading sources from GitHub

For vehicles you also need an extra script in order to let the `ManureSystem` detect your mod.
You will need to get a copy of the latest `ManureSystemVehicle.lua` file which can be found in the [GitHub repository](https://github.com/stijnwop/manureSystem/tree/master/docs/ManureSystemVehicle.lua).

How to download the `ManureSystemVehicle.lua` file:
1. Go to the `docs` folder located in the root directory.
2. Click on the `ManureSystemVehicle.lua` file.
3. A window will open with the script file.
4. Click on the button called `Raw` next to the `Blame` button and it will open the file in RAW format.
5. Right click and click on `save as` (or hit ctrl - s on your keyboard) and save the file to the preferred location in your mod.
 
> REMEMBER: Rename the file extension to `.lua` and don't save it as .txt!

## Adding the ManureSystemVehicle specialization
> In order to start with this step you need to have completed the part `What do I need?`.

> TIP: when you don't plan to add extra specializations besides the ManureSystemVehicle and your mod uses the vehicle type `manureBarrel` you don't necessarily have to add the ManureSystemVehicle.lua as the ManureSystem inserts the specs by default for the vehicle type `manureBarrel` 

### Step 1
Open the `modDesc.xml` file located in your modfolder.

In order to load the specialization you will need to add the specializations entry to the modDesc.
```xml
<specializations>
    <specialization name="manureSystemVehicle" className="ManureSystemVehicle" filename="ManureSystemVehicle.lua"/>
</specializations>
```

The filename must be the exact location of the `ManureSystemVehicle.lua` file you downloaded earlier. In this case it's loaded from the root directory of the mod.

If a similar entry already exists you can just add the specialization entry to that.
```xml
<specialization name="manureSystemVehicle" className="ManureSystemVehicle" filename="ManureSystemVehicle.lua"/>
```
### Step 2
Now we need to add the newly loaded spec to a vehicle type.

In the example below we parent from the vanilla vehicle type `manureBarrel` for our convenience. This can also be something different depending on the vehicle you're adapting.
> NOTE: DON'T BLINDLY COPY PASTE THE PARENT IN THIS CASE AS THIS WON'T SUIT ALL CASES

Here we add the newly registered spec name `manureSystemVehicle`.
```xml
<vehicleTypes>
    <type name="myNewBarrel" parent="manureBarrel" filename="$dataS/scripts/vehicles/Vehicle.lua">
        <specialization name="manureSystemVehicle"/>
    </type>
</vehicleTypes>
```

Copy the new vehicle type name (In our case `myNewBarrel`) and close the modDesc file.

### Step 3
Open the vehicle xml file and change the type="" entry on the second line of the file.
Which looks similar to this:
```xml
<vehicle type="manureBarrel">
```

Rename the manureBarrel to your newly added vehicle type name:
```xml
<vehicle type="myNewBarrel">
```

> Note: if your vehicle uses vehicleTypeConfigurations you also need to change the types there!

#### Step 4
Awesome, we added the `ManureSystemVehicle` to your mod!

## Determine what to add

In order to tell the `ManureSystem` what specializations to add you need to add the following entry to your vehicle XML.
```xml
<manureSystem />
```

We have 4 options/attributes we can set here:

- hasConnectors: `true/false` if the mod has ManureSystem connectors (e.g. couplings, docking funnels)
- hasPumpMotor: `true/false` if the mod has a ManureSystem pump motor
- hasFillArm: `true/false` if the mod has a ManureSystem fill arm (e.g. normal fill arm or docking arm)
- hasFillArmReceiver: `true/false` if the mod allows ManureSystem fill arms to fill from it's fillVolume (e.g. containers)

In this example we're going to add connectors, the pump motor and a fillarm. In order to tell that to the ManureSystem mod we do the following:
```xml
<manureSystem hasConnectors="true" hasPumpMotor="true" hasFillArm="true"/>
```

If your mod also supports to receiver just simply add the `hasFillArmReceiver="true"` attribute.

## Setting up the PumpMotor
> In order todo this step you need to make sure you configured the `hasPumpMotor` entry from the chapter `Determine what to add`.

For the pump motor we have a couple of configuration possibilities.

- litersPerSecond: `250` the liters per second the pump can handle at max efficiency.
- toReachMaxEfficiencyTime: `1000` the time in ms the pump needs to reach the max throughput. (This time is used as a starting point and will be influenced based on the manure thickness or hose length)
- isStandalone: `true/false` determines if the pump functions as a standalone pump (so a pump without capacity on it's own)
- useStandalonePumpText: `true/false` determines if the vehicle should use the standalone pump text, which is different than the standard pump text (left/right instead of in/out)

You also have the options to use a custom sound file for the pump.
An example entry for a standalone pump will be:
```xml
<manureSystemPumpMotor isStandalone="true" litersPerSecond="250" toReachMaxEfficiencyTime="1000">
    <sounds>
        <pump template="SLURRY_02">
            <pitch indoor="0.85" outdoor="0.75"/>
        </pump>
    </sounds>
</manureSystemPumpMotor>
```

Here we tell that it's a standalone pump with the `isStandalone` attribute and with a the pump throughput of 250 liters per second and that it takes 1 second to reach that with the `litersPerSecond` and `toReachMaxEfficiencyTime` attributes. In the example above you can also see the `<sounds>` entry where we use a different template for our pump sound.

An example entry for a normal tanker pump will be:
```xml
<manureSystemPumpMotor litersPerSecond="195" toReachMaxEfficiencyTime="1050"/>
```

As simple as that!

## Setting up the FillArm
> In order todo this step you need to make sure you configured the `hasFillArm` entry from the chapter `Determine what to add`.

For the fill arm we have to following configuration possibilities:

- fillYOffset: `float` e.g. `-0.5` the offset for the fill arm on fillable sources (in order to reach places easier).
- fillUnitIndex: `int` e.g. `1` the fillUnitIndex of which it should fill.
- needsDockingCollision: `true/false` if the fill arm supports docking and needs to required collision for that.

For setting the fillArm node you have the option to create a transform group manually or let the script handle it for you.

Through the entire mod you have the option to create a node with the `createNode` attribute or just refer to an existing node with the `node` attribute.

This options, for creating nodes, comes with the following settings:

- createNode: `true/false`
- When createNode is set to `true`:
    - linkNode: `0>` it's defaulted to the rootNode of your object, but allows setting a custom linkNode for our node to create.
    - position: `0 0 0` the xyz translations of the node to create.
    - rotation: `0 0 0` tje xyz rotations of the node to create.
- When createNode is set to `false`:
    - node: `0>` the index of the node it should use for the fill arm.

An example entry for creating a node through the XML would be:
```xml
<manureSystemFillArm createNode="true" linkNode="armLinkNode" position="-0.888 0.7 3.065" rotation="0 -90 0" />
```

In the example above we let the script create a fill arm node and link it to the given `armLinkNode` and give it a custom position and rotation with the `position` and `rotation` attributes.

You don't have to use this option and have to freedom to choose if you want to refer to an existing node you placed yourself in the i3d file.
> Personally I prefer using the createNode setting which allows for i3d avoidance.

An example entry without the option for creating a node would be:

```xml
<manureSystemFillArm node="armNode"/>
```
In this case the `armNode` will be the node to use for the fill arm, instead of the node we created with the createNode option from above.

When you set the `needsDockingCollision` the `ManureSystem` will load a docking collision on your fillArm.
This is needed in order to allow vehicles with funnels to detect the fill arm.
This collision might not always be on the correct positions as it by defaults takes the same locations as the fill arm node you defined earlier.
If you want to offset the position or rotation of this docking collision you have to add the collision entry.

This will look like this:
```xml
<manureSystemFillArm node="armNode">
    <collision position="0 0.1 0" rotation="90 0 0"/>
</manureSystemFillArm>
```

In most cases it's not needed to set the position or rotation on the collision.