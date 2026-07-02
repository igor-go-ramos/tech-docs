# How to Resolve Error 303: Fan Failure / Overheating
Use this guide when a unit shuts down unexpectedly and displays **Error 303** on the status LED or terminal output.

---

## 🛠️ Required Tools
* T10 Torx screwdriver
* Compressed air can
* Replacement 40mm fan (Part #FN-4020)

## 📋 Step-by-Step Instructions

### Step 1: Safe Shutdown
1. Turn off the main breaker switch on the rear of the unit.
2. Disconnect the AC power cable.

!!! danger "High Voltage Warning"
  Wait **2 minutes** after disconnecting power for the internal capacitors to fully discharge before opening the chassis.

### Step 2: Clear Debris
1. Remove the 4 Torx screws securing the top shroud
2. Inspect the intake grill. If blocked, use compressed air to clear dust, spraying *away* from the optical sensors.

### Step 3: Test or Replace Fan
1. Locate the 3-pin fan connector on the mainboard (labeled `Fan_1`).
2. Disconnect and check for bent pins.
3. If the fan blades do not spin freely by hand, swap the fan out with a new one.

```bash
# Technicians: To clear the error log via serial console, run:
sysctl --clear-faults=303
```

## 🔍 Verification & Expected Results
* Normal behavior: Upon reboot, the status LED should cycle from solid Red to Blinking Green.
* Still Failing? If Error 303 persists after fan replacement, the mainboard thermal sensor has failed. Replace the entire chassis assembly.