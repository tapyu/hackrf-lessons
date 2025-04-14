# HackRF IQ Data Capture and GNU Radio Companion Flowgraph

This repository documents the steps we've taken to capture raw IQ data using a HackRF and process it in GNU Radio Companion (GRC) for visualization. (Installation steps for HackRF and GNU Radio are not included here.)

## Overview

- **HackRF Data Capture:**  
  We used the `hackrf_transfer` utility to capture raw IQ samples. The HackRF outputs IQ samples as interleaved signed 8-bit integers (one byte for I and one for Q).

- **GNU Radio Companion Flowgraph:**  
  We designed a GRC flowgraph that reads the raw IQ file, converts the interleaved 8-bit data into a complex float stream (the internal format used by most GNU Radio blocks), and visualizes its spectrum.

## Capturing IQ Data from HackRF

1. **Verify Device Connection:**
  Connect your HackRF and run:
  ```bash
  hackrf_info
  ```
2. **Capture IQ Samples:**
  Use the following command to capture 1,000,000 samples (at the default sample rate of 20 MS/s) and save them to a file:
  ```bash
  hackrf_transfer -r sample.iq -n 1000000
  ```
  Note:
  - The -r sample.iq option directs HackRF to write the received data into the file sample.iq.
  - The -n 1000000 option tells it to capture one million samples.
  - The resulting file contains interleaved data: the first byte is I₁, the second is Q₁, third is I₂, fourth is Q₂, and so on.

## Processing the IQ Data in GNU Radio Companion

This section explains how to build a flowgraph in GNU Radio Companion (GRC) that converts the raw file into a complex stream and visualizes its spectrum.

1. **Convert the Byte Stream to Complex IQ Data**
 Since the HackRF produces interleaved 8-bit IQ samples (I1, Q1, I2, Q2, …), perform the following processing:
  1. Group Bytes into Pairs:
    - Stream to Vector Block:
    - Drag a Stream to Vector block onto the canvas.
    - Set its Vector Length (or "Num Items") to 2.
    - This groups every two consecutive bytes into a 2-element vector: [I, Q].
  2. Convert Bytes to Floats:
    - Char to Float Block:
    - Add a Char to Float block.
    - Connect the output of the Stream to Vector block to this block.
    - This converts the interleaved 8-bit values (from -128 to 127) into float values.
    - (Optional): You can add a Multiply Const block after this to scale the values (for example, using a constant of 1/128 to normalize the range to approximately [-1, 1]).
  3. Split the 2-Element Vectors into Two Streams:
  - Vector to Streams Block:
    - Drag a Vector to Streams block onto the canvas.
    - Ensure its Vector Length is set to 2.
    - This block splits each 2-element vector into two parallel float streams:
      - Output 0: Contains the first element (I samples).
      - Output 1: Contains the second element (Q samples).
  4. Combine the Float Streams into Complex Samples:
    - Float to Complex Block:
      - Add a Float to Complex block.
      - Connect Output 0 (I stream) from the Vector to Streams block to the Real In port.
      - Connect Output 1 (Q stream) to the Imag In port.
      - This block creates a complex float stream where each sample is represented as I + jQ.
  5. Throttle Block (Optional):
    - For simulation or file playback (when not tied to real hardware), add a Throttle block.
    - Set the Sample Rate parameter to 20e6 (20 MHz) to mimic the HackRF’s sampling rate.
2. Visualize the Signal
  - QT GUI Frequency Sink:
    - Drag a QT GUI Frequency Sink block onto the canvas.
    - Connect the output of the Float to Complex block (or the Throttle block if used) to this sink.
    - Set the Sample Rate to 20e6.
    - Set the Center Frequency to 0 (since the data is now baseband).
    - Adjust autoscale or y-axis limits as needed.
3. Run the Flowgraph
  - Save the flowgraph.

Click the green “Play” button.

You should now see the spectrum display, which represents the frequency content of your captured IQ data.

# TODO: Put here the correct figure
  

