# encled - SCSI Enclosure indicators (SES LED) control

This utility can be used to turn the fault/location leds of an external disk enclosure on and off. The script was tested
only with supermicro shelfs and LSI controllers, but it could in principle also work with shelfs from other manufactures. 

This script is a rewritten version of the original work from George Shuklin, 
which can be foud here: https://github.com/amarao/sdled

Compared to the original script, the new script works also without correctly populated device links in 
`/sys/class/enclosure`. This change was necessary due to Redhat Bug 1394089 (https://bugzilla.redhat.com/show_bug.cgi?id=1394089), 
which was not jet fixed at the time of writing.

## Usage

    $ encled --help
    usage: encled [-h] {status,led} ...
    
    encled - SCSI Enclosure indicators (SES LED) control
    
    optional arguments:
      -h, --help    show this help message and exit
    
    available operations:
      {status,led}
        status      show the status of enclosures or disks
        led         turn on/off the leds of a disk
        
Currently two operations are available `status`, which shows the locations of all disks within the shelf, as well as
the status of the LEDs. The other operation `led` turns LEDs on or off. 

### Status
`status` shows information about the disks and the location of the disks within the shelf. The later is correct for
supermicro shelfs with 24 or 44 disks. The layout of new shelfs may be added by extending the `enclosure_layouts` 
dictionary.

    $ encled status --help
    usage: encled status [-h] [-e ENC] [-d DISK]
    
    optional arguments:
      -h, --help            show this help message and exit
      -e ENC, --enc ENC     the enclosure of interest.
      -d DISK, --disk DISK  the disk of interest.


To get the status of all disks in all enclosures run:

    $ encled status 

    Enclosure: LSI, SAS2X36, 0e12 (/dev/sg24)
    --------------------------------------------------------------------------------
    Slot 6: /dev/sdf    Slot 12: /dev/sdl   Slot 18: /dev/sdr   Slot 24: /dev/sdx   
    35000cca01aa9b2b0   35000cca01aab4818   35000cca01aab0be0   35000cca01aaaa01c   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   
    
    Slot 5: /dev/sde    Slot 11: /dev/sdk   Slot 17: /dev/sdq   Slot 23: /dev/sdw   
    35000cca01aa9aa58   35000cca01aab5ec0   35000cca01aab110c   35000cca01aa9a830   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   
    
    Slot 4: /dev/sdd    Slot 10: /dev/sdj   Slot 16: /dev/sdp   Slot 22: /dev/sdv   
    35000cca01aab559c   35000cca01aaae048   35000cca01aab558c   35000cca01aa8b2e8   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   
    
    Slot 3: /dev/sdc    Slot 9: /dev/sdi    Slot 15: /dev/sdo   Slot 21: /dev/sdu   
    35000cca01aaaeccc   35000cca01aab56c8   35000cca01aa6e7cc   35000cca01aa79c10   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   
    
    Slot 2: /dev/sdb    Slot 8: /dev/sdh    Slot 14: /dev/sdn   Slot 20: /dev/sdt   
    35000cca01aaaeb18   35000cca01aa9437c   35000cca01aa71a80   35000cca01aab4838   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   
    
    Slot 1: /dev/sda    Slot 7: /dev/sdg    Slot 13: /dev/sdm   Slot 19: /dev/sds   
    35000cca01aab63e4   35000cca01aaaeaa4   35000cca01aab4434   35000cca01aab48e4   
    Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   Status=OK L=0 F=0   

For one individual disk run:

    $ encled status --disk /dev/sdx
    
    Enclosure: LSI, SAS2X36, 0e12 (/dev/sg24)
    Slot 24: /dev/sdx
    35000cca01aaaa01c
    Status=OK L=0 F=0
    
The `--disk` argument takes names like `/dev/sda`, the SCSI-ID (which is also used by the multipath kernel module), or
an integer representing the slot (counting from 1!).

If multiple enclosures are present, the output may be reduced to one enclosure by adding the `--enc` Argument. It
Expects the name of the SCSI-device associated with the enclosure (e.g., `/dev/sg24`).


### LED
`led` turns leds on and off again:
    
    $ encled led --help         
    usage: encled led [-h] [-e ENC] [-d DISK] {locate,fault,off}
    
    positional arguments:
      {locate,fault,off}    select the led to turn on/off.
    
    optional arguments:
      -h, --help            show this help message and exit
      -e ENC, --enc ENC     the enclosure of interest.
      -d DISK, --disk DISK  the disk of interest.

The difference between `locate` and `fault` depends on your specific hardware. The supermicro shelf used for testing had
only one LED per disk for fault and locate. The `locate` command makes it blinking, the `fault` command turns it on 
continuously.

To turn on all LEDs of a specific enclosure run:

    $ encled led --enc /dev/sg24 locate
    
To locate one specific disk run:

    $ encled led --disk 35000cca01aa9437c locate

The `--disk` flag understands the same arguments as it does within the `status` operation.
