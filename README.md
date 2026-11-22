# SimpleCPU
This is a simple CPU that can follow a list of 16 instructions and perform addition, subtraction, and simple logic operations on 2-bit numbers. I made this to learn how to use gates, and it is a good project for people who have experience with hardware but not with gates to try.

It works in several steps. First, the user flips switches in a matrix to manually code the instructions. Then, the user presses run, and those switches tell the ROM, another matrix, what the instructions are. Then, the user powers on the CPU with a switch and resets the counter. Then, the clock starts pulsing 1s and 0s to the counter. The counter then increments by one every clock tick, or jumps a specific amount in certain cases. Next is a decoder that, given the number instructions from the counter, selects a row in the ROM, each of which hold an instruction. The row is inputted into the ALU, which performs the selected operations. Sometimes the instruction will want to use the value from the previous operation, so the accumulator stores the values from the previous operation. Then there is a selector that decides if the ALU gets a new value or the previous output value. The ALU outputs flags, such as negative and zero, which essentially describe the output. The next section then decides whether the program should just go to the next instruction or jump to a different instruction based on those flags. It tells this to the program counter, and the program counter moves to the next part. Then it all repeats.

## Clock
<img width="837" height="314" alt="Screenshot 2026-11-09 at 5 18 47 PM" src="https://github.com/user-attachments/assets/9fd6f162-591f-4e8c-b2b4-249e1f4149ad" />

This clock is made of 3 not gates, which invert the value put into them. So, if the input is 1, the output is 0. Because there are an odd number, their output value alternates between 0 and 1 every pass. The 1 is essentially a clock tick. The switch gives power the clock; without it, it can't run. Finally, the and gate is there to only output a value every tick, since it only outputs 1 when the clock value is 1.

## Program Counter (PC)
The job of the PC is to track what step we are on. It moves to the next step or jumps (I'll explain this later) every clock tick.


<img width="362" height="589" alt="Screenshot 2026-11-21 at 7 48 41 PM" src="https://github.com/user-attachments/assets/c5d19326-c650-4bbc-a9dd-ba8b2a29f4ba" />

The first part of the PC is a multiplexer (mux) that first the number of step we were just on, and a number from the Read-Only Memory that is a step to jump to. It then decides whether it should increment the step we're on by 1 or jump based on an input from the branch logic part of the CPU.


<img width="432" height="780" alt="Screenshot 2026-11-21 at 7 52 08 PM" src="https://github.com/user-attachments/assets/0056b132-4dd8-41ce-bc1b-15c9178e142b" />

The second part of the PC is an adder. It takes the output from the mux (either 1 for "go to the next step" or another number for a jump) and adds that number to the number of the previous step. It works by using XOR gates, AND gates, and OR gates.


<img width="238" height="787" alt="Screenshot 2026-11-21 at 7 53 31 PM" src="https://github.com/user-attachments/assets/dd4fe64d-af38-44bc-a26d-f01b7a066ed3" />

That last part is a register (flip flops that store the value). This keeps the number step it is on (a 4 bit number) using 4 slave-master latches (1 for each bit). It also feeds this value back into the adder part of the PC as the value that the increment or jump is added to.

## Read-Only Memory (ROM)
The point of the ROM is the store the instructions to run, and then using the number step from the PC, access the number instruction and send it to the rest of the CPU to run.


<img width="414" height="801" alt="Screenshot 2026-11-21 at 7 57 26 PM" src="https://github.com/user-attachments/assets/b0ba651a-699a-4ee0-b147-acbd49524567" />

The first part of it is a decoder that gets the output from the PC. If that output is 0110 (6), for example then the decoder makes all of its outputs high except for the 6th output. This means that the CPU is going to run the 6th instruction stored in the ROM.


<img width="709" height="792" alt="Screenshot 2026-11-21 at 8 00 42 PM" src="https://github.com/user-attachments/assets/ee12e358-516a-4cee-9231-5263894a260f" />

The ROM stores all of its instructions in this 16x15 matrix. Every row is an instruction, and each column is a different part of that instruction needed to make a full instruction. The matrix is made up of latching switches. They have two coils so that you can turn them on and off electronically, and then they hold their value even while the CPU is off.


<img width="479" height="639" alt="Screenshot 2026-11-21 at 8 02 28 PM" src="https://github.com/user-attachments/assets/016aef6e-9226-48dd-b512-942c98e83a1e" />

This is one latch in the matrix. It is the first one in it's row. The row is connected to the NOT gate seen on the left, since as I said earlier the decoder turns all rows high except the one that is chosen, which it turns low. This not gate makes the chosen row high and the others low, which is needed for the CPU to work. It connects to pin 10 on the latch, which is the input pin. The +R and +S coils in this latch are connected to diodes so that their current doesn't leak. The diodes are connected to a row in the assembler (which I'll explain later) so that the latch's value can be changed. Pin 9, the output pin, is connected to a diode so that current doesn't leak to other lines. Then the diode is connected to a wire that runs all the way down the column. Every latch in the same column has a value for the same type. For example, the type could be the 2nd bit of the input value A, or the address that the PC will jump to. At the bottom of every column is a 10k resistor connected to GND.

## Arithmetic Logic Unit (ALU)
The ALU is the heart of the CPU. It does all the computations that are needed for the program. The values for the computation are stored in the ROM, as well as the operation to do to those values. It can only take in 2 value that are each 2 bits.


<img width="789" height="798" alt="Screenshot 2026-11-21 at 8 13 28 PM" src="https://github.com/user-attachments/assets/a60ab219-baef-461e-91de-41a448aaec86" />

This part of the ALU is the adder and subtracter. It takes in the 2 2 bitvalues, denoted a A0, A1, B0, and B1 in this image, and adds or subtracts them based on another input. It works by using XOR gates, AND gates, and OR gates. The AND gate are the top of the image decides whether this adder/subtracer should add or subtract. The AND gates takes two inputs, S0 and S1 (I'll explain where they come from later), and outputs a 1 or 0 to say add or subtract. This output is used as the input for some of the XOR gates. This section of the ALU can both add and subtract, since for subtracting, it inverts the second number, adds it to the first, and adds 1.


<img width="133" height="800" alt="Screenshot 2026-11-21 at 8 19 56 PM" src="https://github.com/user-attachments/assets/b0fc567f-6582-4bfe-bc48-8bc8b720703d" />

This part of the ALU does 4 operations: it can see if values are both 1, see if values are different, see if at least one of the values is 1, or see if none of the values are 1. It uses AND gates, XOR gates, OR gates, and NOT gates respectively to do these different operations.


<img width="873" height="737" alt="Screenshot 2026-11-21 at 8 21 56 PM" src="https://github.com/user-attachments/assets/d3ef6476-8c43-44b9-ad52-12ef1b58816f" />

Since there are many operations that the ALU can do, these two muxs decide which operation to use. The ALU does all of the operations I talked about above the input numbers, but these muxs only let the outputs from one of the operations through. There are 2 muxs, since the inputs are 2 bit numbers and each mux can only deal with one bit. They take in all the outputs from the different operations and also S0, S1, and S2 from the ROM. The S's tell the muxs which value to let through. If those three input values are 010 (2), the muxs let the second value through. I6 on both of them is not being used, so it is tied to GND. They output F0 and F1, which go the the flags, LEDs, and accumulator.

## Flags
Often times the CPU doesn't care what the output number is; it only cares about its characteristics (negative, zero, etc.) These flags describe those characteristics. There are 3 flags: negative, zero, and overflow. Negative sees if the output is negative, zero sees if the output is zero, and overflow sees if the output was more than 2 bits (this could happen if 11 and 11 were added, for example, as that would result in 110).


<img width="501" height="548" alt="Screenshot 2026-11-21 at 8 27 39 PM" src="https://github.com/user-attachments/assets/c2b5d2ef-ea08-41a0-b324-fb1a3ab69b7f" />

These gates determine those flags about the output. Note that there are only two gates, for overflow and zero, since the negative flag is simply the output COUT from the adder. The gate for the zero flag is an NAND gate that sees if both F0 and F1 are zero. If they are, it outputs 1, saying that the output from the ALU was 0. The overflow XOR gate takes the COUT output from the adder and the output of the first OR gate in the adder and sees if they are different. If they are, it outputs 1, which means the output from the ALU was more than 2 bits.

## LEDs
<img width="303" height="733" alt="Screenshot 2026-11-21 at 8 34 49 PM" src="https://github.com/user-attachments/assets/487b0027-6b16-4591-980d-a9af3c075278" />

These LEDs are a visual output that show both the flag values and the number. From top to bottom, they represent these values: zero flag, negative flag, overflow flag, first digit of output, second digit of output, and third digit of output or a negative sign if the value is being read as signed.

## Accumulator
<img width="473" height="724" alt="Screenshot 2026-11-21 at 8 36 21 PM" src="https://github.com/user-attachments/assets/ec126e0c-5feb-4e17-87f9-590986002976" />

The accumulator stores the outputs from the ALU in 2 master-slave latches (one for each of the 2 output bits). It takes the output value from the ALU and clock (CLK) as its inputs so it stores the output value and doesn't change it while the clock is in the middle of a tick, since that would mess up its value.

## A Select
Sometimes, we want to do an operation to 2 new values, but sometimes we want to do an operation to an old value, like in a for loop.

<img width="641" height="699" alt="Screenshot 2026-11-21 at 8 42 50 PM" src="https://github.com/user-attachments/assets/1c31dc26-1fe3-4c11-8592-c64a5a19a97b" />

Because of this, we have a mux that decides if we want to use a new value for A0 and A1 or the old one. It takes the old value from the accumulator and a new value from the ROM. It also takes a value from the ROM called A_OPCODE, which determines if the output of the mux is the old or new value. Based on that, it outputs a 2 bits that are sent to the ALU as the A0 and A1 values.

## Branch Logic
Sometimes, we want to skip to a different instruction based on the output of the last operation.


<img width="749" height="716" alt="Screenshot 2026-11-21 at 8 44 52 PM" src="https://github.com/user-attachments/assets/33d0d499-1d8a-49a0-afb5-e0de1a358b80" />

To decide this, we use a mux and an XOR gate. The mux takes in the flag values and ties all the other unused inputs to GND. It also takes in two bits from the ROM called FLAG0 and FLAG1. These 2 values tell the mux what flag to look at. If those values make the number 01 (1), the mux looks at the first flag. It outputs the value of that flag to the XOR. The XOR then takes another input from the ROM called FLAG_VALUE. FLAG_VALUE says if the program should jump to a different step if the falg value is 0 or 1. Then it outputs JMP to the PC mux, which I mentioned before.

## Set Clock

## Set Counter

## Assembler

## Reset Switch
