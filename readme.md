<p align="center">
    <img height="50%" width="50%" src="https://raw.githubusercontent.com/nahiyan/miden/master/miden-logo.png">
    <p align="center">An assembler for 16-bit RISC processors.</p>
</p>

# Introduction
Miden is an assembler that assembles MIPS assembly to machine code for 16-bit RISC processors.

# Instruction Set Architecture
Miden supports 7 instructions and 8 registers. You can find more information [here](https://github.com/nahiyan/miden/blob/master/docs/isa.md).

# Example

Here's a C program which multiplies two numbers:

```cpp
int main() {
    int y;
    y = multiplyNumbers(2, 3); /*Returns product of 2 and 3*/
}

int multiplyNumbers(int a, int b) {
    int i;
    int product = 0;
    for (i = 0; i != b; i = i + 1) {
        product = product + a;
    }
    return product;
}
```

It's equivalent to the following piece of assembly code:

```asm
main:
	li $s0, 0
	li $a0, 2
	li $a1, 3
	jal multiplyNumbers
	addi $s0, $v0, 0
	j done

multiplyNumbers:
	li $t0, 0
	li $v0, 0
	j multiplyNumbersLoop

multiplyNumbersLoop:
	beq $t0, $a1, multiplyNumbersReturn

	add $v0, $v0, $a0
	addi $t0, $t0, 1

	j multiplyNumbersLoop

multiplyNumbersReturn:
	jr $ra
done:
	
```

This assembly program to fed into Miden. Miden generates machine code out of it.

Here's the machine code generated out of the assembly code above:

```binary
0110010000000000
0111000000000010
0111010000000011
1010000000001100
0010010110000000
1000000000011100
0110100000000000
0110110000000000
1000000000010010
0100101010011010
0000110110000100
0010100100000001
1000000000010010
1100000000000110
```

# How Miden Works
Summarizing everything, Miden takes the assembly code and generates machine code out of it.

Here are the steps involved in the process:

## Step 1: Tokenize
The first step in the process is to convert the assembly code into series of tokens.

To do so, the assembly code is fed into a tokenizer. The tokenizer reads the assembly code and converts it into a series of tokens.

Let's say you've got 2 lines of assembly code:

```asm
main:
	li $s0, 0
```

If you feed the code into the tokenizer, you'll get 4 tokens:

```
LBL 	main
KWD 	li
REG 	s0
INT 	0
```

### Tokens
Each token has:

- a name
- a value

As an example, the first token has a name of "LBL" and a value of "main."

Token names are 3 characters long for simplicity.

You can read more about tokens [here](https://github.com/nahiyan/miden/blob/master/docs/tokens.md).

As you can see from the assembly code, we've got:

- a label (LBL) "main"
- a keyword for an instruction (KWD) "li"
- a register (REG) "s0"
- an integer (INT) "0"

You can get an idea of how the tokenizer divides the assembly code into identical chunks, or as we know - tokens.

### Labels

The tokenizer identifies the labels as seen above. It doesn't stop there, it remembers the labels and keeps log of the instruction address each label points to.

> In the example above, the label named "main" points to the first instruction named "li," so it has address of 0.

## Step 2: Parse
Next step in the process is to take the tokens generated and feed it to the parser. The parser will generate machine code from the tokens.

Now that we have converted the assembly code into a series of tokens, it's much easier to understand to go through the tokens one by one, understand the meanings using simple logic and generate machine code!

As seen from the example code in step 1, we've got 4 tokens to parse:

```
LBL 	main
KWD 	li
REG 	s0
INT 	0
```

Here are the steps the parser will take to generate machine code:

- The parser will ignore the label declaration as the tokenizer already took care of logging it down.
- The parser smartly identifies the keyword, "li," as an instruction. This will trigger it to expect a register and an immediate value in the next 2 tokens:
	- Register "s0" and immediate integer value "0" are identified

- Parser, after successfully identifying an "li" (load immediate) instruction, generates a machine code out of it.

Here's the final output:

``
0110010000000000
``

### Machine Code
Let's break down the machine code that has just been generated:

``
0110010000000000
``

Since Miden generates machine code for 16-bit processors, machine codes of each instruction is, as expected, 16 bits long.

This machine code is for:

``
li $s0, 0
``

which is an instruction of type "RI." More on this in the [ISA](https://github.com/nahiyan/miden/blob/master/docs/isa.md).

Since it's an RI-type instruction, here's the formatting:

- The first 3 bits "011" represents the opcode for "li."
- The next 3 bits "001" represents the binary for register "s0."
- The next 10 bits represent the immediate value "0."
