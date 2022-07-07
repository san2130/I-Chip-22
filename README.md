# Verilog Implementation of A5/1 GSM Crytographic Cipher  

## About A5/1 Cryptographic Algorithm
- A5/1 is a symmetric stream encryption algorithm (cipher) which is hardware-based and is used for confidentiality in GSM cell phones.  
- It is called hardware based because it can be implemented using the instruction set on computer architecture. 
- It employs **three Linear Feedback Shift Registers (LFSRs)** X, Y, and Z.  
The three LFSRs X, Y and Z have lengths equal to 19, 22, and 23 bits respectively. (Note: 19 + 22 + 23 = 64 bits)   
<p align="center">Register X can be represented as (x0, x1, x2, …., x18)</p>
<p align="center">Register Y can be represented as (y0, y1, y2, …., y21)</p>
<p align="center">Register Z can be represented as (z0, z1, z2, …., z22).</p>  

- A **64-bit private key** K and a **22-bit public key** are used as the initial values in X, Y, and Z registers. After the registers are filled with the keys, the keystream is generated through these three registers. The keystream is then **XORed** with the input bitstream to get the encrypted stream. 
- For more details on generating the entire keystream refer to [this](https://www.cryptographynotes.com/2019/02/symmetric-stream-a5by1-algorithm-linear-feedback-shift-register.html)  
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/reg.jpg"/></p>
<br>

## Modifications for enhancing image encryption security 
- We break the image into **8 bit-planes**. Each plane contains 256 x 256 bits.
- For example, let our image have two pixels with first pixel=8’b0111_0011 and second pixel=8’b0001_0011 then the first plane will contain only 2’b00, second plane will contain 2’b10, third again 2’b10 and so on. Similarly for other pixels. 
- Thus, for a 256*256 image, our image will be actually broken into 8 planes of 256 x 256 pixels each.  
- Now we simply encrypt the image plane by plane. But to make the cipher strong, before the start of a new plane, we **re-initialize** the LFSRs following the same steps mentioned earlier with the **public key's last 4 bits as plane number(1-8)**. ([Detailed Problem Statement](https://github.com/san2130/I-Chip22/blob/main/I_CHIP-PS2.pdf)) 
<br>
 <p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/ps.jpg" width="70%"/></p>  
 <br>

## Verilog Design Approach 
Vivado Xilinx Design Suite was used to implement and simulate the circuit design. ([Detailed Report](https://github.com/san2130/I-Chip22/blob/main/Report.pdf))  
- First the (256*256) grayscale image is converted to a **hexadecimal** file using a MATLAB [script](https://github.com/san2130/I-Chip22/blob/main/ps22022.mlx). 
<br>
 <p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/conv1.jpg" width="50%"/></p>  
 <br>
 
- The contents of the hexfile is read in the testbench and stored in
an array of 8 bit registers ‘mem’.
<br>
 <p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/test1.jpg" width="50%"/></p>   
<br>

- Now for the first **65536** positive clock edges the first bit from
the right of every member of ‘mem’ is stored in a register of
65535 bit size p1.
- Next the public key is generated, that is by taking the first 18
bits of the chosen public key and appending the plane number
of the current plane. Plane numbers go from 1 to 8
- Now the design module is **activated** and the inputs are the
public key, private key, p1 containing the first bit of every
pixel, clock and trigger tr. tr=1 activates the design module.
<br>
 <p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/design1.jpg" width="50%"/></p> 
<br>

- Now in the design module, the LSFRs are initialized with
zeros, the first **64** clock cycles are used feeding the **private key**
and the next **22** are used feeding the generated **public key**.
- Then **100** clock cycles with **majority logic** ignoring outputs and
the next **65535** cycles generating the **final cipher key** for the
plane.
- Now p1 is fed to the design module in **batches** of **256** bits since
passing all the 65535 bits together caused memory issues.
- For every such input batch an answer of 256 bits is produced
by **xoring** the input with the cipher key. This answer is then
stored in the testbench in a register of 65535 bits.
- Once the entire plane has been processed, we have an
encrypted plane and the bits are then **reassembled** and stored in
the final output memory.
- Then all the LSFRs are **reinitialized**, everything is reset and
the next plane is processed with the public key ending with the
new plane number.
- Once all the planes are processed the output register
contents are written to the output file.
<br>
 <p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/conv2.jpg" width="50%"/></p>
<br>

<h2 align="center">Simulation and Results</h2>  
 
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/input.jpg" width="30%"/><br><i>Input (256*256) Image</i>
</p>  
<br>
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/darr.png" width="10%"/></p>
<br>
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/output.jpg" width="30%"/><br><i>Encrypted (256*256) Image</i></p>
<br>
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/darr.png" width="10%"/></p>
<br>
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/output2.jpg" width="30%"/><br><i>Decrypted (256*256) Image</i></p>
<br>
<br>
<p align="center"><img src="https://github.com/san2130/I-Chip22/blob/main/media/waveform.jpg" width="100%"/><br><i>Simulation Waveform</i></p>
<br>
<br> 

<h2 align=center>Our Team</h2>
    
<table align=center>
   <td align="center">
      <a href="https://github.com/san2130">
         <img src="https://avatars.githubusercontent.com/u/88130555?v=4" width="150px;" alt=""/>
         <br />
         <sub>
            <b>Sandeepan Ghosh</b>
         </sub>
      </a>
      <br />
   </td>
   <td align="center">
      <a href="https://github.com/sanyam-dev">
         <img src="https://avatars.githubusercontent.com/u/56957500?v=4" width="150px" alt=""/>
         <br />
         <sub>
            <b>Sanyam Jha</b>
         </sub>
      </a>
      <br />
   </td>
</table>
