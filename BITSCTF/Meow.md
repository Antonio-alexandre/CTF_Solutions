


# MEOW TRANSFORMATION: CTF WRITE-UP

----------

## Approach

The challenge presented a $128 \times 128$ pixel image of a cat containing hidden metadata. The metadata described a "journey through chaos" with three distinct "leaps," "styles," and "rhythms." This terminology pointed directly to **Arnold's Cat Map**, a chaotic transformation where pixel coordinates are shuffled but eventually return to their original positions due to the map's periodicity.

The recovery process required mathematically "unwinding" the image through three stages of matrix transformations and then performing a deep-dive into the bit planes to find a hidden steganographic layer.

### Step 1: Decoding the Metadata

The intercepted string provided the exact parameters needed to reverse the encryption:

-   **Dimensions ($N$):** $128 \times 128$ pixels.
    
-   **Styles ($p$):** $[1, 2, 1]$ (The shear parameter of the matrix).
    
-   **Current Iterations:** $[47, 37, 29]$ (How many times the agents spun the image).
    
-   **Full Periods ($T$):** $[96, 64, 96]$ (The number of spins required to return to "zero").
    

The transformation follows the Hamiltonian matrix logic:

$$\begin{bmatrix} x' \\ y' \end{bmatrix} = \begin{bmatrix} 1 & p \\ 1 & p+1 \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} \pmod N$$

### Step 2: Precise Reverse Transformation

To restore the image, we had to complete the cycles. If an image is spun $k$ times and the total period is $T$, applying $T - k$ more spins returns it to the original state. We reversed the "adventure" sequence (Last-In, First-Out):

1.  **Stage 3:** Applied $96 - 29 = 67$ spins with Style $p=1$.
    
2.  **Stage 2:** Applied $64 - 37 = 27$ spins with Style $p=2$.
    
3.  **Stage 1:** Applied $96 - 47 = 49$ spins with Style $p=1$.
    

We utilized a Python script with `NumPy` for the coordinate mapping and `Pillow` for image rendering.

### Step 3: Bit-Plane Analysis

Once the cat image was restored to its "pre-scrambled" state, the pixels appeared normal, but the flag remained hidden. Because the Cat Map shuffles pixel positions, any **LSB (Least Significant Bit)** encryption is also shuffled and unreadable until the image is restored.

We imported the restored image into **StegSolve** and cycled through the bit planes. The flag was revealed with perfect clarity in the **Red Plane 0**, appearing as white text against a black background.

----------

## Flag

Plaintext

```
BITSCTF{4rn0ld5_c4t_m4ps_4r3_p3r10d1c}

```

----------

## Tools Used

-   **Python 3 (NumPy & PIL):** Used to script the custom matrix iterations.
    
-   **ExifTool:** To extract the hidden parameters from the original image metadata.
    
-   **StegSolve:** To perform the bit-plane walk and extract the final LSB-hidden flag