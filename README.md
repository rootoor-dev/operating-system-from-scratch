# Creating an Operating System (OS) from scratch
Creating oneself an Operating System (OS) has still been the dream of all noobs, but for nerds too. Everytime we try to make ours, we are discouraged by pessimists. Today morning, I decided to make my own OS from scratch with clear and simple explanations of the steps I will follow to do it. Feel free to contribute and teach others.

# The beginning : steps

Firstly, our OS has to print "Hello, World!" to start.

1. **Hello World**: Start by creating a program that displays "Hello World," which requires about 27 lines of code due to hardware specifics.

2. **Understanding the BIOS**: Familiarize yourself with the BIOS, the software that runs at startup and configures the computer's hardware.

3. **Creating the Bootloader**: Develop a bootloader that must be exactly 512 bytes and end with a specific binary signature to be recognized by the BIOS.

4. **Switching to Protected Mode**: Modify the CPU's operation to switch from real mode, which is limited, to protected mode, allowing for more advanced memory management.

5. **Configuring the Global Descriptor Table (GDT)**: Establish a GDT to define memory segments and privilege levels, ensuring the OS's security.

6. **Loading the Operating System**: Load the operating system code using a cross-compiler to ensure it is suitable for the targeted architecture.

7. **Displaying on Screen**: Interact with the graphics card to display characters on the screen using VGA text mode.

8. **Managing the Cursor and Scrolling**: Implement a visible cursor and manage scrolling when the user reaches the end of the screen by directly manipulating input-output ports.

# Details

The first step is always a hello world when you’re learning a new programming language. That’s how you start—by displaying Hello World on your screen. Well, we’re going to do the same for an OS, or operating system, but saying OS is just quicker, sorry to the academics. An OS that just displays Hello World, and that’s it. Yep, that’s it! So hurry up! Haven’t you watched the video or something? Usually, doing a hello world is literally one line of code, but here it’s 27 lines.

You'll need to read a bunch of documentation to really get what's happening when your computer starts up, but don’t worry—I’ll sum it up for you. When you power on your computer, the first program that runs is the BIOS, or UEFI if you have a newer machine, but I’ll focus on the BIOS. This little piece of software is stored on a chip on your motherboard, and it’s all pre-coded by the manufacturer. There’s nothing you need to do; the BIOS is responsible for identifying and configuring your computer’s hardware once it starts up.

He's done, he's starting the test run which checks that everything's working fine and shows you the results on the screen. If all's good, he’ll want to hand over commands to our OS, but how does he know where our OS is, what size it is, or which function?

The advantage of the bootloader is that we can create it by following very strict specifications defined by the BIOS, which will allow it to be easily found by it because yes, exploring all four of my SSDs in search of a boot code is not viable. To avoid this, we will place our bootloader code in the boot sector—a single address that the BIOS will check to see if our bootloader is there or not; it's much simpler. But how does it know that there is indeed bootloader code there and that it's not just random data? Well, thanks to two properties: the bootloader code must be exactly 512 bytes and will always end with a very specific binary signature. So our BIOS analyzes the boot sector of each storage device available on the computer and checks if there is a 512-byte segment ending with our signature; if so, it knows it's the bootloader. It then loads its code into memory more precisely at address 0x7C00 and executes it from the very first bit.

Okay, let’s code this but in what language? Well, you guessed it; yeah I know but it's the language that translates most directly into binary for our machine and it's the only one that meets all the required characteristics—there's no choice. Let's start with the end with the binary signature; for that we just need to define it and yes this number is indeed our signature but we've just written it in hexadecimal. Now let's make sure that the file is exactly 512 bytes.

This last instruction takes two bytes so we have 510 left to fill; we will calculate the size of our code from the current position to the first line and see how many bytes we are missing. For this missing number, we will repeat adding zeros. So if we look at the binary file created from our assembly we have our signature, our code which will be at the beginning and between them we've added plenty of zeros to reach 512 bytes.

Now we just need to write something on the screen; how do you remember from BIOS? The fastest way to do this is with its help—maybe we shouldn't have insulted it after all. I told you that BIOS displayed test results on screen so that means it has a function to display a character on screen and it turns out we can ask it to use this function for us. This request is made via an interrupt; an interrupt consists of asking our CPU to stop what it's doing to execute a priority request then return to what it was doing before.

0x10 is the interrupt that allows us to ask the CPU to use BIOS's video display service and to communicate our exact request to BIOS we will pass values via CPU registers. A register is just a small place where we can store data; more precisely by storing value 0E in register AH we ask to display a character on screen. We could also have asked to display a pixel or change cursor position for example and in register AL we store the character we want to display on screen.

So we pass our variables by modifying register values then call the interrupt; CPU takes control looks at what interrupt corresponds and hands over control to BIOS which then looks at which function we want to use by retrieving value from register AH and now when we want to display a character it looks for value in register AL to know which one and does it—there you go!

Now let's add an infinite loop—a piece of code that CPU will execute continuously without stopping—why? Because CPU has a work ethic my friend; it never stops working—it wants to execute instructions nonstop and if it reaches end of our bootloader code then it will execute whatever comes next in memory but that's random values that mean nothing so it'll crash like an idiot but if we trap it in our infinite loop then it won't crash.

And there we are—we displayed 'H' on screen; that's nice but we're missing 10 letters. Well, we could just copy-paste and that's that but doing so would be disgusting—0 out of 10—but there's potential and reaching it requires making a function that executes for each character of our phrase "hello world". Eight minutes—we spent eight minutes explaining a hello world; I think you understand now what kind of mess we're in.

So now it's time—we're finally going to boot our OS—no wait! We must first switch to protected mode! CPU manufacturers are obsessed with backward compatibility—they want that if you take an ancient BIOS and run it on your latest CPU everything works perfectly fine—and for this we must emulate an ancient CPU.

Wait—I think I just insulted part of my audience there—yeah no no no—not you! Time passes faster in technology—it’s well known—it’s fine no no no okay so when we boot our CPU behaves as if it's from 1978—that's called real mode—in which for example less than 1 MB of RAM is available—1 MB! We can't stay like this; it's up to us to advance time and ask our CPU to stop its cosplay.

We will do this by switching into protected mode which will be more than sufficient for us—and we'll have hold onto your hat—up to 4 GB of RAM! And this protected mode will also allow us—as its name indicates—to protect our OS because in real mode there's zero protection—an application can do whatever it wants access all memory all resources use any imaginable instructions—in short with protected mode we'll be able restrict external applications at different privilege levels called rings.

Ring 0 will be used by our OS and will have most freedom—no restrictions—while a regular application will be in ring 3 with very limited access—and if ever an app needs access to a resource protected for example it'll have to request permission from our OS which will decide what happens next.

So we need set rules—and these rules are ours define them and give them CPU—we do this with Global Descriptor Table or GDT—as its name indicates—it’s a table allowing us define different segments within memory; each entry defines one of these segments—we give starting address size then some properties like whether this section can be read modified executed etc.—and thus everything is secured.

Interesting right? Well who cares? Yeah this segmentation story isn’t used much anymore—nowadays we do paging which we'll need set up later B if there's time—but still we must define GDT—it’s mandatory again because backward compatibility—we'll create GDT with actually no segmentation—that's called Basic Flat Model which is described in Intel's user manual.

This involves integrating three entries into GDT—the three entries which are actually mandatory—we start with null descriptor with all bits zero then define segment containing code and segment for data—the only difference being code can be read executed but not modified while data can be read modified but not executed—in both cases they get max privilege with ring zero—and make sure they occupy all available space in memory so they actually overlap—they're superimposed as said earlier—we don't want segmentation that's why!

And there you go—we have our table just need note where it starts as well as its size and give this info CPU so knows where find it. Now that we've defined GDT we'll switch into protected mode—for this just disable interrupts—indeed interrupts are currently managed by BIOS except BIOS only works in real mode—we can't use anymore—we'll have handle interrupts ourselves later.

Then all that's left is change value register CR0 of our CPU—as soon as this value changes CPU knows it's in protected mode and should no longer operate in real mode—and finally there's one last little problem: CPUs are ultra fast—and for being efficient they use what's called pipelining—in short when CPU executes line of your code—it does so in several steps: first loads instruction then decodes know exactly how process it with its various components then executes—to go as fast as possible—it can do these steps in parallel across multiple instructions from your code.

Let’s take an example: CPU starts loading first line of your code when it's done starts decoding while loading next line—in once first line decoded CPU executes while decoding second line just loaded while loading third line—the problem here is if switch into protected mode—it’s possible subsequent instructions meant execute protected mode actually execute real mode since got ahead with pipelining—and that's problematic!

We must prevent pipelining—and do this we'll perform long jump—we'll make distant jump into our code—CPU won't know what next instruction until has made jump thus won't load won't pipeline—it'll have wait—and we'll be sure it'll be in protected mode!

Now that we're in protected mode we'll finally load our OS—and this OS we'll code in C! If someone had told me one day I'd celebrate coding in C... well—to test our code we'll make ultra simple OS with just infinite loop like what did for hello world OS back at beginning video—that's C code allowing us do so!

We want compile transform into binary executable—but there's small problem—yeah there's always small problem when coding low level—it breaks compiler notes—it’s designed create binary code Windows 11 on my machine 64 bits with all system functions etc.—except this code we're going execute not Windows but on this here architecture totally broken which has nothing defined!

If compile using current compiler it'll crash—we must create what's called cross-compiler—we'll create compiler producing binary code suitable for OS we're coding! Okay sounds complicated but really isn’t—you just download C compiler source found online give parameter architecture want use suited coding then run—that creates special compiler for system with which compile code!

Okay great but also need call OS code from bootloader—the problem being don’t know where OS code is! Let’s solve linking OS code with bootloader creating bootable disk image combining both codes making sure OS comes right after bootloader's code located in boot sector—as mentioned beginning video!

We can now load OS code from storage device using previously defined BIOS function like did for hello world except this time using interrupt 13! Once executed we've loaded code at chosen address RAM simply because sure it'll be free zone once switched protected mode—all that's left call this code job done!

We get black screen as expected where can see reflection slowly realizing spent hours researching coding just get damn black screen! Well let’s add features now because it's kinda light like this already would be nice display text screen oh well easy just use BIOS function like did before—but no wait! 

BIOS only works real mode while we're protected mode—we'll have manage without! Don’t worry—not complicated screen managed graphics card when booting it's VGA mode old graphic resolution again due backward compatibility—but VGA has text mode that'll save us time since we're broke!

Text mode has built-in font everything needed—all simply tell graphics card which character display where—more precisely VGA text mode works grid 80 x 25 characters each character defined by two bytes: first byte ASCII code second byte properties displayed screen stored address 0xB8000!

So if point address set first byte letter E second byte value 0x0F get white 'E' at very start screen—from there displaying character anywhere easy: row times number columns per row plus column where want display character—and voila printed character printed phrase hello world version two my friend!

But we're not done managing screen having little cursor indicating current position useful especially when want user type commands—you might imagine simply displaying little character grid done—but nope every time counterintuitive grid characters communicated graphics card via RAM while cursor communicates directly through input-output port linking CPU graphics card more precisely want communicate cursor position!

But there's small problem yes another small problem always small problems position takes 16 bits while input-output port can only transmit 8 bits at once! So 16 bits too much doesn’t fit—to solve issue we'll actually use two ports: port 0x3D4 indicates graphics card sending high bits cursor position then via port 0x3D5 send low bits indicating sending last eight bits boom sent now cursor easily updated every time write screen!

Last important point we'd like scroll when reach end screen yes must do everything ourselves! To achieve when reach last line simply shift all characters grid down one...

# Codes of the bootloader

Here’s the code for a simple bootloader that displays "Hello World" on the screen:

```assembly
; Bootloader code to display "Hello World"
section .text
    global _start

_start:
    ; Set up the video mode (text mode)
    mov ax, 0x0003  ; Set video mode to 80x25 text
    int 0x10        ; BIOS interrupt to set video mode

    ; Display "Hello World"
    mov si, message  ; Load address of the message
.next_char:
    lodsb            ; Load next byte from [si] into al
    cmp al, 0       ; Check for null terminator
    je .done        ; If zero, we are done
    mov ah, 0x0E    ; BIOS function to write character
    int 0x10        ; Call BIOS to display character
    jmp .next_char  ; Repeat for next character

.done:
    jmp .done       ; Infinite loop to keep the program running

section .data
message db 'Hello World!', 0  ; Null-terminated string
```

### Explanation:
- **Video Mode Setup**: The bootloader sets the video mode to 80x25 text using BIOS interrupt `int 0x10`.
- **Display Loop**: It loops through each character in the string "Hello World!" and uses `int 0x10` with function `0x0E` to display each character on the screen.
- **Infinite Loop**: After displaying the message, it enters an infinite loop to keep the program running.

This code should be assembled and placed in the boot sector of a disk image for testing.

# Simple bootloader

To create a simple bootloader in assembly, follow these steps:

1. **Set Up the Bootloader**: Your bootloader must be exactly 512 bytes and end with a specific binary signature. This allows the BIOS to recognize it during the boot process.

2. **Write the Code**: Use assembly language to write the bootloader code. The first instruction should set the video mode to text mode using BIOS interrupt `int 0x10`.

3. **Display "Hello World"**: Load the string "Hello World!" into memory and use a loop to display each character on the screen by calling BIOS interrupt `int 0x10` with the appropriate registers.

4. **Infinite Loop**: After displaying the message, implement an infinite loop to prevent the CPU from executing random data that could lead to a crash.

5. **Compile and Link**: Assemble your code into a binary file, ensuring it is exactly 512 bytes by padding with zeros if necessary.

6. **Create a Bootable Disk Image**: Combine your bootloader code with any operating system code you wish to load, ensuring that the OS code follows immediately after the bootloader in the disk image.

7. **Test Your Bootloader**: Use an emulator or a virtual machine to test your bootloader and ensure it displays "Hello World" as expected.

Here’s an example of a simple bootloader code in assembly:

```assembly
; Simple Bootloader Code
section .text
    global _start

_start:
    ; Set video mode to 80x25 text
    mov ax, 0x0003
    int 0x10

    ; Display "Hello World!"
    mov si, message
.next_char:
    lodsb
    cmp al, 0
    je .done
    mov ah, 0x0E
    int 0x10
    jmp .next_char

.done:
    jmp .done

section .data
message db 'Hello World!', 0
```

### Explanation of Key Parts:
- **Video Mode**: The BIOS interrupt `int 0x10` is used to set the video mode.
- **Character Display**: Each character is displayed using a loop that reads from the message string.
- **Infinite Loop**: The program enters an infinite loop after displaying the message to keep running without crashing.

This code should be assembled and placed in a bootable disk image for testing.


















