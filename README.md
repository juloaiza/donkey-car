# donkey-car
Donkey-car

#### PS4 Controller

The following instructions are based on [RetroPie](https://github.com/RetroPie/RetroPie-Setup/wiki/PS4-Controller#installation) and [ds4drv](https://github.com/chrippa/ds4drv).

#### Install `ds4drv`.

Running on your pi over ssh, you can directly `sudo /home/pi/env/bin/pip install ds4drv`

#### Grant permission to `ds4drv`.

```bash
sudo wget https://raw.githubusercontent.com/chrippa/ds4drv/master/udev/50-ds4drv.rules -O /etc/udev/rules.d/50-ds4drv.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```
#### Run `ds4drv`.

`ds4drv --hidraw --led 00ff00`. If you see `Failed to create input device: "/dev/uinput" cannot be opened for writing`, reboot and retry. Probably granting permission step doesn't take effect until rebooting. Some controllers don't work with `--hidraw`. If that's the case try the command without it. `--led 00ff00` changes the light bar color, it's optional.



#### Start controller in pairing mode.

Press and hold **Share** button, then press and hold **PS** button until the light bar starts blinking. If it goes **green** after a few seconds, pairing is successful.

#### Run `ds4drv` in background on startup once booted.

`sudo nano /etc/rc.local`. paste `/home/pi/env/bin/ds4drv --led 00ff00` into the file. Save and exit. Again, with or without `--hidraw`, depending on the particular controller you are using.


To disconnect, kill the process `ds4drv` and hold **PS** for 10 seconds to power off the controller.


### Troubleshooting

Check here for frequently encountered
issues.

#### Failed to create input device: "/dev/uinput" cannot be opened for writing

This could be because the uinput kernel module is not running on your
computer. Doing `lsmod | grep uinput` should show if the module is
loaded. If it is blank, run `sudo modprobe uinput` to load it. (The
uinput module needs to be installed first. Please check with your
distro's package manager.)

To have the uinput module load on startup, you can add a file to
`/etc/modules-load.d`. For example:

``` bash
# in file /etc/modules-load.d/uinput.conf
# Load uinput module at boot
uinput
```

## Install on donkeycar

1. Connect your bluetooth controller to the raspberry pi. See the Bluetooth section below.

2. Install the parts python package.
    ```bash
    pip install git+https://github.com/autorope/donkeypart_ps3_controller.git
    ```

3. Import the part at the top of your manage.py script. If you are using a PS4 controller, import `PS4JoystickController` instead. Same applies in the next step.
    ```python
    import donkeypart_ps3_controller.part as ps4
    ```   

4. Replace the controller part of your manage.py to use the JoysticController part.
    ```python
    ctr = ps4.PS4JoystickController(
       throttle_scale=cfg.JOYSTICK_MAX_THROTTLE,
       steering_scale=cfg.JOYSTICK_STEERING_SCALE,
       auto_record_on_throttle=cfg.AUTO_RECORD_ON_THROTTLE
    )

     V.add(ctr,
          inputs=['cam/image_array'],
          outputs=['user/angle', 'user/throttle', 'user/mode', 'recording'],
          threaded=True)
    ```

5. Add the required config paramters to your config.py file. It should look something like this.
    ```python
    #JOYSTICK
    JOYSTICK_STEERING_SCALE = 1.0
    AUTO_RECORD_ON_THROTTLE = True
    ```

6. Now you're ready to run the `python manage.py drive --js` command to start your car.
