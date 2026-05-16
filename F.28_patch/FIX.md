# F28 Patch
**Disclaimer**
I am not an expert in ACPI or DSDTs, this came from a lot of testing and use of AI. These edits may work for you as they worked for me, but I'm really in no position to help anyone.

## Firstly, make the edits from F.27 provided by no-hands.
*I would recommend adding comments like `// Edit #n`, as you will have to jump back to these locations, so its far easier to navigate by searching for edits.*

## Edit #1
At the location of Edit #3 for F.27, you need to make these changes
```
 Method (_INI, 0, NotSerialized)  // _INI: Initialize
        {
            /* [Edits] */
            // Change #3
            
            // New to F28
            INT1 = GNUM (GPDI) 
            INT2 = INUM (GPDI)
            // End to F28

            // Set the "enabled" bit (I2CN) to true
            Or (\_SB.PC00.I2C0.I2CN, One, \_SB.PC00.I2C0.I2CN)
            // Steer firmware to ELAN07CA path on 0x15 at 400kHz, using GPIO IRQ
            Store (0x05, \TPDT)    // branch that leads to ELAN mapping in your table
            Store (0x15, \TPDB)    // 7-bit I2C address 0x15
            Store (One,  \TPDH)    // ELAN subselector
            Store (One,  \TPDS)    // speed selector: 0=100k, 1=400k, 2=1MHz → use 400k
            Store (Zero, \TPDM)    // 0 => use SBFG (GPIO) in _CRS, not GSI -> this handles interrupts

            // If firmware left placeholder HID, force ELAN vendor HID
            If (LEqual (_HID, "XXXX0000"))
            {
                Store ("ELAN07CA", _HID)
            }
            /* [End Edits] */
```

## Edit #2
At the location of Edit #4 from F.27, make these changes
`Store (0x02, \TPDS)` needs to be changed to `Store (One, \TPDS)`


## Additional edits
These seem to conclude the important changes. A few lines above the change to Edit #3 (_was line 107610 for me_), at the line
`GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,`, I had changed `Level` to be `Edge`, but it seems the compiler switched it back. So for completeness try this if the other changes are still not working.


## GPU and Power Saving
On my system at least, the Nvidia GPU wasn't fully sleeping. So even if I wasn't using it, it was pulling a ton of power (around 55-60 W for me). The work around I found was using `envycontrol` and `powerprofilesctl` to setup aliases for power states. Switching my gpu off and setting to power-saver when I didn't need the Nividia gpu, I could drop my energy consumption to about 21-24 W. Not a required fix but worth mentioning for anyone trying to make Linux a daily driver.


*Good luck. If you have any comments, fixes, or similar, please make an issue or discussion on the repository. More eyes and voices in this issue will help the community.*

