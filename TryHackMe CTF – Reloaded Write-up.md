## TryHackMe CTF – Reloaded Write-up

Developed by WhiteHeart and tested by IslaMukheef

What I used for these tasks:

- Ghidra for static analysis
- x32dbg for debugging purposes
- Strings/FLOSS (last task)

## Level 0 – Find the flag

We are given file **Level-0_1610287551028.exe**, let’s run it.

The application prompts us for a flag, let’s try to find it.

File seems not packed nor obfuscated so let’s put the file into Ghidra and analyze the Defined Strings.

After a quick glance I found a string that might be the flag, lets try to input it into the argument.

That seemed to be the flag.





## Level 1 – Find the flag

Let’s analyze it in Ghidra and find the string **Try again**.

I found the function that compares the input with param_1, which is written in HEX, converting it into decimal seems to be the flag for this level.

## Level 2 – Find the flag

Running the file **Level-2_1610288072323.exe**

Let’s search for the output strings in Ghidra.


We have few declared variables and if statement that depends on the previously used strmp.

If the string comp is equal to 0, we will get the to the flag. Assembly code uses Jump if not zero (JNZ) to do that (x0040144d). 

If we reverse the logic by simply changing it to Jump if Zero (JZ) then we will reach the statement despite the wrong input and potentially get the flag.

Now after changing the instruction let’s patch the file and run it again.






## Level 3 – Find the flag

For this task we are required to download the file ‘Level-3_1610295976945.exe’.

Let’s run it like previous ones.

Let’s put it into Ghidra, and look for the ‘Enter the flag ::’ to see what’s going on.


Based on the screenshot I can tell that it scans for the input, holds it in the ‘local\_24’ and calls FUN\_004014af function. Let’s take a look at it.

We have few declared variables, a for loop, if statement that depends on the previously used strmp.

If the string comp is equal to 0, we will get the ‘Rooted !!!’ prompt. Assembly uses Jump if not zero (**JNZ**) opcode to do that (**x0040150a**). But there’s no point of that since there is no flag. 

I think the flag might be stored in a local_20 which is compared with our input.

We need to set a breakpoint before the strcmp, let’s use a call to **FUN_00401410(local_28)** located at **x004014bc** as our breakpoint.

After few memory step-overs we can see our flag stored in EAX. 

## Level 4 – Find the flag

File: **Level-4_1610450489985.exe**


As a result we get unknown string, decoding it with online tools did not result in any success.

Let’s analyze it with Ghidra.

To find more about the unknown string we need to search for the method **printf**. 

Since it’s encoded, we can’t find the string like in the previous tasks.

**Looks like we have a stack string that each ASCII char is being XOR’ed by either 0x37 or incremented iStack_10 variable.**

Let’s run ollydbg and set a breakpoint at x00401453 before XOR function to see if we can find the flag.

After few steps we get a string stored in EAX registry.


There is also other method to do this task, it is much faster and doesn’t require knowledge about disassembling and debugging.

The method is using the tool ‘FLOSS’ which stands for The FLARE Obfuscated String Solver created by Mandiant.  It searches for static strings, stack strings, tight strings and decoded functions.

More information on this can be found at: <https://github.com/mandiant/flare-floss>

To execute it we simply enter:

`floss Level-4_1610450489985.exe`

The floss has decoded the stackstring and that is our flag.



To check if the string is a decoded message I decided to code a short script in C.

And the result for the input ‘Alan’ is:

Which is the start of four letters in encoded message we have seen earlier:





