# cpus-pdp8
This is a branch of Brad Parker's TSS-8 revival efforts, with the goal of modifying TSS-8 to run entirely off an RK05 pack -- requiring no RF08/RF32 disk for swapping or system files.  Files unrelated to TSS-8 have been removed from this branch.

Source files for this port are in tss8_lcm2.  They can be built using the assembler in the utils/macro directory.
An RK05 disk image containing a running system is provided in utils/maker/tss8_rk_lcm.dsk.

Hardware requirements:
----------------------
A PDP-8 system with:
  - Memory Extension and Timesharing option (with Timesharing option enabled)
  - RK8E or equivalent and at least one RK05 drive.
  - 12KW of core, minimum
  - Line-time clock or programmable clock
 
SIMH Setup:
-----------
Under simh, the following script will set up the hardware and bootstrap TSS-8 INIT:

set df disabled
set rf disabled
set rk enabled
set dt disabled
att rk0 tss8_rk_lcm.dsk

set cpu 32k
attach ttix 4000

load boot.bin
run 200

Running on real hardware:
-------------------------
A tool like Dave Gesswein's DUMPREST (ftp://ftp.pdp8online.com/software/dumprest/) can be used to write out the tss8_rk_lcm.dsk file to a real RK05 pack.  The initial bootstrap (tss8_lcm2/boot.lst) must be toggled into the front panel and started at address 200.  This will bring INIT into core and execute it.

As long as core is intact, subsequent reboots can be accomplished by restarting the PDP-8 at address 3053.  Ensure that AC, IF, and DF are initialized to zero before restarting.  


Starting TSS-8 and logging in:
------------------------------
Once started, INIT will print the following:

  LOAD, DUMP, START, ETC?

Answer "START" (or just "S") and you will be prompted for the date and time.  Choose a date between 1974 and 1984.

  MONTH-DAY-YEAR: 11-25-79
  HR:MIN - 11:46

Hit Return and you will be at the monitor's dot prompt (".").  The only command accepted at this point is a LOGIN command.  This command will not be echoed.

The following accounts are set up on tss8_rk_lcm.dsk:

  1 - System Manager - Password: 1234
  2 - Librarian - Password: 1234
  1000 - Guest account - Password: LCML

Once logged in, you will be greeted with a banner, and will be at the monitor prompt. 

  TSS/8.24+  JOB 01  [00,01]  K00    11:45:16

  WELCOME TO THE LCM+L TSS/8 SYSTEM.
  .

Several programs are at your disposal ("R CAT" while logged in under the Librarian account will show you what's available):

  .R CAT

  DISK FILES FOR USER  0, 2 ON 25-NOV-79

  NAME      SIZE  PROT    DATE
  PALD  .SAV  16   12  31-MAR-76
  LOADER.SAV   4   12  31-MAR-76
  FORT  .SAV   6   12  31-MAR-76
  FOSL  .SAV   6   12  31-MAR-76
  PIP   .SAV  10   12  31-MAR-76
  TSTLPT.SAV   2   12  31-MAR-76
  LOGOUT.SAV   6   12  31-MAR-76
  SYSTAT.SAV   5   12  31-MAR-76
  EDIT  .SAV   8   12  31-MAR-76
  FOCAL .SAV  16   12  31-MAR-76
  BASIC .SAV  38   12  31-MAR-76
  COPY  .SAV  10   12  31-MAR-76
  CAT   .SAV   6   12  31-MAR-76
  GRIPE .SAV   5   12  31-MAR-76
  LOGID .SAV   4   12  31-MAR-76
  PUTR  .SAV  21   12   3-FEB-84
  ODTHI .SAV   2   12  29-FEB-84
  FLAP  .SAV   1   12   7-APR-84
  PTLOAD.SAV   1   12  29-APR-84
  BLANK .SAV   1   12   9-JUN-84
  DTTEST.SAV   2   12  26-JUN-84
  INIT  .SAV  17   12  29-JUL-84
  HELPR .BIN   3   12  21-NOV-79
  HELP        17   12  21-NOV-79
  CHESS .SAV  17   12  11-NOV-74
  ALGOL .SAV  32   12  21-NOV-79
  CHEESE.BAS  19   12  21-NOV-79
  ACTUNG.SAV   5   12  21-NOV-79
  LISP  .SAV  17   12  25-NOV-79

  TOTAL DISK SEGMENTS:  297    QUOTA: 1575


Reconfiguring the system
------------------------
tss8_rk_lcm.dsk was built for a PDP-8/e/f/m system with 32KW of core, a TC08 dectape controller, a high-speed paper-tape reader and 8 KL8E asynchronous serial lines.  If your PDP-8 has different a different hardware configuration than this, you may want to reconfigure and rebuild TSS-8.  (This is required if you have less than 32KW of core or have a different CPU type.  Nonexistent hardware doesn't generally cause issues, but it can free up extra core for TSS-8 structures.)

Most of the configuration is in tss8_lcm2/melrose.pal -- this is fairly straightforward:  set the parameters to correspond with your hardware.

To adjust the amount of core in your system, set CORMEM to the appropriate value -- 70 for 32KW, 20 for 12KW, etc.  Additionally, a small change to the "maker" program in utils/maker needs to be made:  The line

  fields[2][01403] = 6; /* CORFLD */
  
in field_patch() must be modified to the number of user-available core fields on your system.  For a 32KW system this is 6 (TSS-8 takes up two fields, leaving 6 available).  On a 12KW system this is 1.

Once these changes are made, go to the next section

Rebuilding ths system
---------------------
To rebuild the TSS-8 code, do a "make clean; make" in tss8_lcm2.  If there are any errors during assembly these will be reported -- check any .err files generated and make corrections as necessary.

Once the TSS-8 binaries are rebuilt, they need to be loaded into a disk image -- the tools for doing so are in utils/maker.  This tool can be rebuilt (if need be) by again doing a "make clean; make".  Then the tss8_rk_lcm.dsk image can be updated with new binaries and patches via:

  maker lcm
  
The resultant disk image can then be run in simh or on real hardware.

Limitations
-----------
Obviously, running a system from an RK05 is going to be slower than with an RF08 due to the seek times involved.  Filesystem operations under extreme conditions may be considerably slower, especially if software does many repeated write operations of small amounts of data.

At this time, it is not possible to run the system from RK05 and support RK05 drives as assignable devices.  This should not be impossible but requires changes to the original RK05 device code and the new RK05 system code to ensure they don't stomp on each other.









