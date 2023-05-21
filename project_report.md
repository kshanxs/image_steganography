# Secure Image Steganography using Least Significant Bit (LSB) Substitution and Cryptographic Encryption

**Course**: Final Year Project (B.Tech Computer Science and Engineering)  
**Status**: Completed  

---

## Abstract
Image steganography is the art and science of concealing secret messages within cover images to protect the confidentiality of information during transmission. While standard Least Significant Bit (LSB) steganography is simple and efficient, it suffers from a major security loophole: any observer aware of the algorithm can easily extract the hidden plaintext message. 

To overcome this vulnerability, this project implements a **hybrid security system** that combines cryptography and steganography. Before embedding, the secret message is encrypted using AES symmetric-key cryptography (Fernet protocol) with a key derived from a user-supplied password via SHA-256 hashing. The encrypted ciphertext is then embedded into the carrier image's pixel values using the LSB technique. Finally, a user-friendly web interface is built using Gradio to enable seamless encoding and decoding. The final application shows that the steganographic changes are completely imperceptible to the human eye, providing a robust, dual-layered solution for secure communication.

---

## 1. Introduction & Problem Statement
With the rapid growth of digital communication, securing sensitive data during transmission has become a critical challenge. Cryptography is widely used to encrypt messages, making them unreadable to unauthorized parties. However, the presence of encrypted data itself can attract suspicion from attackers or censors. 

**Steganography** addresses this problem by hiding the very existence of the message. By embedding the message into a cover medium (such as an image), the communication is kept secret. 

### The Problem
Most basic steganography implementations embed text directly as plaintext. If an adversary suspects steganography is being used, they can run simple extraction scripts to retrieve the plaintext message immediately. 

### The Solution
This project designs and implements a dual-layer security protocol:
1. **Cryptographic Layer**: The message is encrypted with a password-based key using the Fernet symmetric encryption standard.
2. **Steganographic Layer**: The resulting ciphertext is hidden inside the LSBs of the carrier image.
3. **Interactive UI**: A local and web-deployable interface is provided to make the tool accessible.

---

## 2. Theoretical Background & Methodology

### 2.1 Least Significant Bit (LSB) Steganography
Digital images are represented as grids of pixels. In standard 24-bit RGB images, each pixel consists of three color channels (Red, Green, Blue), and each channel is stored as an 8-bit byte (values from 0 to 255).

The LSB is the lowest bit in a binary representation. For example, if a pixel's red channel value is $145$ (binary `10010001`), the LSB is `1`. Changing the LSB from `1` to `0` changes the value to $144$ (binary `10010000`). This minor shift changes the color intensity by only $\frac{1}{256}$, which is completely imperceptible to the human eye.

### 2.2 Algorithm Flow

#### A. Encoding Flow
1. **Key Derivation**: The user inputs a plaintext password. The password is hashed using **SHA-256** to create a 32-byte key.
2. **Encryption**: The secret message is encrypted using **AES-128 in CBC mode (Fernet)**, generating a base64-encoded ciphertext string.
3. **Header Addition**: The length of the ciphertext is calculated and converted into a 32-bit binary representation. This header is embedded first so the decoder knows how many bytes to read.
4. **LSB Substitution**: The ciphertext bits are sequentially written into the LSBs of the image's pixel channels (R, G, and B) sequentially.
5. **Output**: The output image is saved as a lossless PNG to prevent compression from corrupting the embedded bits.

#### B. Decoding Flow
1. **Header Extraction**: The first 32 bits are read from the LSBs of the image to determine the length $L$ of the encrypted payload.
2. **Payload Extraction**: The next $L \times 8$ bits are extracted from the LSBs and reconstructed into the ciphertext byte array.
3. **Decryption**: The user enters the password. The key is derived using SHA-256. If the password is correct, the ciphertext is decrypted back to the original secret message. If the password is incorrect, the system catches the cryptographic failure and alerts the user.

---

## 3. System Architecture & Implementation

The application is written in Python and is divided into two primary modules:
1. **Algorithm Engine (`LSBSteg`)**: Manages pixel-level read/write operations using OpenCV and NumPy.
2. **User Interface (`Gradio`)**: Manages the web server, uploads, and downloads.

### 3.1 Core LSB Embedding Logic
The custom class `LSBSteg` handles the manipulation of the NumPy image matrix:
```python
class LSBSteg():
    def __init__(self, im):
        self.image = im
        self.height, self.width, self.nbchannels = im.shape
        self.size = self.width * self.height
        ...
```
The method `put_binary_value` modifies individual bits using bitwise OR and bitwise AND masks, preventing corruption of other bits:
* **To set a bit to 1**: `val | maskONE` (where `maskONE = 1`)
* **To set a bit to 0**: `val & maskZERO` (where `maskZERO = 254` or `11111110`)

### 3.2 Cryptographic Implementation
Symmetric encryption is handled via Python's `cryptography` library. A key derivation helper creates a secure key from the password:
```python
def generate_key(password):
    return base64.urlsafe_b64encode(hashlib.sha256(password.encode()).digest())
```
By utilizing SHA-256, any input password of arbitrary length is securely hashed into a fixed-size 256-bit key required for Fernet encryption.

---

## 4. UI Design & Deployment

The application features a responsive browser-based UI using **Gradio**, which allows users to interact with the software without running code in the terminal.

### 4.1 Interface Layout
The UI is divided into two primary tabs:
* **Encode Tab**:
  * Inputs: Carrier Image (Drag and Drop), Secret Textbox, Password (hidden characters).
  * Outputs: Encoded Image (downloadable file).
* **Decode Tab**:
  * Inputs: Encoded Image, Password.
  * Outputs: Decoded Text (output textbox).

### 4.2 Web Deployment
The system was successfully deployed on **Hugging Face Spaces** using Gradio, hosting the application in a cloud environment accessible via a standard web browser.

---

## 5. Testing & Experimental Evaluation

### 5.1 Capacity Calculation
The maximum message capacity of the carrier image is calculated as:
$$\text{Max Capacity (Bytes)} = \frac{\text{Width} \times \text{Height} \times \text{Channels}}{8} - 4$$
*(Note: 4 bytes are subtracted for the 32-bit length header)*

For a standard $1920 \times 1080$ Full HD image:
$$\text{Max Capacity} = \frac{1920 \times 1080 \times 3}{8} - 4 = 777,596 \text{ Bytes} \approx 777 \text{ KB}$$

This is more than sufficient for storing long text files or scripts.

### 5.2 Visual Imperceptibility
Testing was performed by encoding a 500-word paragraph into a $512 \times 512$ cover image. 
* **Visual Inspection**: Side-by-side comparison of the cover image and the steganographic image revealed no visible differences in color, shading, or sharpness.
* **Histogram Analysis**: The red, green, and blue histograms of the original and encoded images showed near-identical distributions, demonstrating that statistical stegananalysis would fail to detect the payload.

---

## 6. Conclusion & Future Work
This final year project successfully demonstrates a highly secure, dual-layered image steganography tool. By encrypting the message before embedding, the solution guarantees confidentiality even if the steganography itself is compromised. The Gradio interface provides a modern and accessible frontend, and the Hugging Face hosting proves its cloud readiness.

### Future Scope
1. **Adaptive Steeganography**: Hiding data only in high-contrast/texture regions of the image to further reduce detectability.
2. **Video Steganography**: Extending the LSB algorithm to hide data inside the frames of an MP4/AVI video file.
3. **Deep Learning Steganography**: Using Generative Adversarial Networks (GANs) to generate natural-looking cover images optimized for data hiding.
