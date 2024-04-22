## TryHackMe CTF – Reloaded Write-up

Developed by WhiteHeart and tested by IslaMukheef

What I used for these tasks:

- Ghidra for static analysis
- x32dbg for debugging purposes
- Strings/FLOSS (last task)

## Level 0 – Find the flag

We are given file **Level-0_1610287551028.exe**, let’s run it.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/56f72827-9017-4545-90fd-70db0edb9edf)


The application prompts us for a flag, let’s try to find it.

File seems not packed nor obfuscated so let’s put the file into Ghidra and analyze the Defined Strings.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/0a148d22-47ee-40d1-9745-0556fbc650f4)


After a quick glance I found a string that might be the flag, lets try to input it into the argument.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/64a2949b-cdeb-43e2-80f0-88cdc8bc041a)


That seemed to be the flag.





## Level 1 – Find the flag

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/6f8ed865-49d8-4c09-ab98-2277a296ed18)

Let’s analyze it in Ghidra and find the string **Try again**.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/e3109351-77ba-4693-b32c-f8c297dd80f6)


I found the function that compares the input with param_1, which is written in HEX, converting it into decimal seems to be the flag for this level.



## Level 2 – Find the flag

Running the file **Level-2_1610288072323.exe**

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/982d4c18-f32a-41c9-8fb4-21b64aa7b4ce)


Let’s search for the output strings in Ghidra.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/24b4f7d4-638f-44bf-9b3a-57d307a19051)

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/6e6b3bcb-7e71-48a2-881a-65453279e00b)



We have few declared variables and if statement that depends on the previously used strmp.

If the string comp is equal to 0, we will get the to the flag. Assembly code uses Jump if not zero (JNZ) to do that (x0040144d). 

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/78d4e56c-7d75-479a-bcd0-b51d2a8e67fc)


If we reverse the logic by simply changing it to Jump if Zero (JZ) then we will reach the statement despite the wrong input and potentially get the flag.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/84f22d0b-6ee6-45a6-963c-cbfbdb5df28b)


Now after changing the instruction let’s patch the file and run it again.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/00ad17d5-fbc2-425c-a961-821e199ed3aa)





## Level 3 – Find the flag

For this task we are required to download the file ‘Level-3_1610295976945.exe’.

Let’s run it like previous ones.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/b6d102c1-a3db-47e9-b57e-4016cd3d3b20)


Let’s put it into Ghidra, and look for the ‘Enter the flag ::’ to see what’s going on.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/6fa210ed-8c1c-4895-945c-f82708d02db8)


Based on the screenshot I can tell that it scans for the input, holds it in the ‘local\_24’ and calls FUN\_004014af function. Let’s take a look at it.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/f3bde1b9-e00b-4875-8bf7-9c826be0f149)


We have few declared variables, a for loop, if statement that depends on the previously used strmp.

If the string comp is equal to 0, we will get the ‘Rooted !!!’ prompt. Assembly uses Jump if not zero (**JNZ**) opcode to do that (**x0040150a**). But there’s no point of that since there is no flag. 

I think the flag might be stored in a local_20 which is compared with our input.

We need to set a breakpoint before the strcmp, let’s use a call to **FUN_00401410(local_28)** located at **x004014bc** as our breakpoint.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/0660625f-6aa0-4aaa-9e58-e448333fe1d2)


After few memory step-overs we can see our flag stored in EAX. 

## Level 4 – Find the flag

File: **Level-4_1610450489985.exe**

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/0b01ff9e-a7ce-4aae-b728-017c0b8c3a44)



As a result we get unknown string, decoding it with online tools did not result in any success.

Let’s analyze it with Ghidra.

To find more about the unknown string we need to search for the method **printf**. 

Since it’s encoded, we can’t find the string like in the previous tasks.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/d8de7221-e6de-457b-8098-400b9ea57674)


**Looks like we have a stack string that each ASCII char is being XOR’ed by either 0x37 or incremented iStack_10 variable.**

Let’s run ollydbg and set a breakpoint at x00401453 before XOR function to see if we can find the flag.

After few steps we get a string stored in EAX registry.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/ab73a951-6afa-4a95-a700-e8df28cfefc6)



There is also other method to do this task, it is much faster and doesn’t require knowledge about disassembling and debugging.

The method is using the tool ‘FLOSS’ which stands for The FLARE Obfuscated String Solver created by Mandiant.  It searches for static strings, stack strings, tight strings and decoded functions.

More information on this can be found at: <https://github.com/mandiant/flare-floss>

To execute it we simply enter:

`floss Level-4_1610450489985.exe`

The floss has decoded the stackstring and that is our flag.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/4877f975-4184-4e21-953d-8334fbfedc36)


To check if the string is a decoded message I decided to code a short script in C.

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/0e979b95-61a2-4b57-8e0b-a36d7ad6389b)


And the result for the input ‘Alan’ is:

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/d14e6c1f-e932-4857-967d-28918a897614)


Which is the start of four letters in encoded message we have seen earlier:

![image](https://github.com/tomaszstopnicki/tryhackme-ctf-writeups/assets/163318997/c9b4ab72-7d32-4c8a-8348-c848f1b9e4b4)





