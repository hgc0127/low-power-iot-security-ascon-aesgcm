# Low-Power IoT Security: Energy Comparison of ASCON and AES-GCM

## Video Link

🔗 https://youtu.be/BpkBcQrJ3DM

---

## 1. Introduction

Low-power IoT devices are used in many environments such as smart homes, healthcare systems, and environmental monitoring. These devices usually run on limited battery power, so energy efficiency is one of the most important design requirements. At the same time, IoT communication must be protected from threats such as data leakage, packet interception, and unauthorized access.

This project focuses on the relationship between **security and energy consumption** in constrained IoT networks. We compared ASCON, a lightweight authenticated encryption algorithm, with AES-GCM, a widely used authenticated encryption method. The experiment was designed around an **IEEE 802.15.4 + 6LoWPAN** environment, where packet size, computation cost, and fragmentation can directly affect energy usage.

Our main question was:

> Which encryption approach is more suitable for low-power IoT communication when payload size increases?

---

## 2. Problem Definition

In low-power IoT networks, security cannot be considered separately from hardware and network limitations. Even if an encryption algorithm is secure, it may not be practical if it increases computation time, memory usage, or packet overhead too much.

This project considers three major constraints:

* **Limited hardware resources**: Sky motes use MSP430-based 16-bit architecture.
* **Small packet size**: IEEE 802.15.4 supports a maximum frame size of 127 bytes.
* **Network overhead**: 6LoWPAN may cause fragmentation when packets become too large.

Because encryption adds authentication tags, padding, and processing overhead, it can increase the total amount of transmitted data. In a constrained network, this may lead to fragmentation and retransmission, which eventually increases energy consumption.

---

## 3. Algorithms Compared

| Algorithm | Main Purpose                             | Relevance to IoT                      |
| --------- | ---------------------------------------- | ------------------------------------- |
| ASCON     | Lightweight authenticated encryption     | Designed for constrained environments |
| AES-GCM   | General-purpose authenticated encryption | Widely used and standardized          |

AES-GCM is commonly used in modern secure communication systems, but it is not always optimized for small embedded devices. ASCON, on the other hand, was selected as the winner of the NIST Lightweight Cryptography project and is designed with constrained devices in mind.

However, our experiment showed that the result is not simply “ASCON is always better.” The energy efficiency depends on payload size, implementation method, and hardware support.

---

## 4. Experimental Environment

The experiment was conducted using Contiki-NG and Cooja simulation.

| Item               | Setting                             |
| ------------------ | ----------------------------------- |
| Simulator          | Cooja                               |
| OS/Framework       | Contiki-NG                          |
| Mote type          | Sky mote                            |
| MCU architecture   | MSP430, 16-bit                      |
| Network stack      | IEEE 802.15.4 + 6LoWPAN             |
| Routing            | RPL                                 |
| Payload sizes      | 16 bytes, 32 bytes, 64 bytes        |
| Measurement method | RTIMER_NOW() and repeated execution |

The encryption tests were repeated multiple times, and the results were averaged to reduce measurement noise.

---

## 5. Implementation Experience

### 5.1 ASCON Porting Limitation

During the implementation process, we first tried to port the actual ASCON reference code to the MSP430 environment. However, the build failed because the MSP430 GCC 4.7.4 toolchain did not properly support the 64-bit operations required by ASCON.

This was an important finding because it showed that lightweight cryptography is not only about algorithm design. Even if an algorithm is designed for constrained environments, the actual hardware architecture and compiler support can still become practical barriers.

Because of this issue, a simplified ASCON-like implementation was also tested to observe relative timing behavior in the same environment.

### 5.2 Simplified Implementation Result

| Algorithm            | Payload Size | Clock Ticks (x100) |
| -------------------- | ------------ | ------------------ |
| ASCON (simplified)   | 16 bytes     | 12                 |
| ASCON (simplified)   | 32 bytes     | 24                 |
| ASCON (simplified)   | 64 bytes     | 49                 |
| AES-GCM (simplified) | 16 bytes     | 4                  |
| AES-GCM (simplified) | 32 bytes     | 9                  |
| AES-GCM (simplified) | 64 bytes     | 17                 |

In the simplified test, AES-GCM appeared faster than ASCON. However, this result should not be interpreted as a final algorithm comparison because the simplified code does not fully represent the internal structure of either real algorithm. Instead, this part of the experiment was useful for understanding the limits of the simulation and porting environment.

---

## 6. Actual Algorithm Comparison

After the initial porting issue, the actual ASCON and AES-GCM implementations were measured and compared using 16-byte, 32-byte, and 64-byte payloads.

| Algorithm | Payload Size | CPU Ticks | Energy (mJ) |
| --------- | ------------ | --------- | ----------- |
| ASCON     | 16 bytes     | 3.271     | 0.00431     |
| ASCON     | 32 bytes     | 3.552     | 0.00468     |
| ASCON     | 64 bytes     | 4.451     | 0.00587     |
| AES-GCM   | 16 bytes     | 15.22     | 0.00250     |
| AES-GCM   | 32 bytes     | 29.007    | 0.00477     |
| AES-GCM   | 64 bytes     | 59.14     | 0.00974     |

### Key Observation

```text
16 bytes: AES-GCM uses less energy than ASCON.
32 bytes: ASCON and AES-GCM show almost similar energy consumption.
64 bytes: ASCON becomes more energy-efficient than AES-GCM.
```

The result shows that AES-GCM performs better for very small payloads, but its energy consumption increases more sharply as the payload size grows. ASCON shows a more gradual increase, making it more suitable for larger IoT sensor data in this experiment.

---

## 7. Visualization

### Energy Consumption Bar Chart

![Energy Consumption Bar Chart](results/energy_bar.png)

### Energy Consumption Line Chart

![Energy Consumption Line Chart](results/comparison.png)

### Average Energy with Standard Deviation

![Average Energy with Std Dev](results/avg_std.png)

The graphs show that the energy gap changes depending on payload size. At 16 bytes, AES-GCM has an advantage. At 64 bytes, ASCON becomes more efficient. The 32-byte point helps show the transition between these two cases.

---

## 8. Network Layer Analysis

Encryption overhead affects not only CPU computation but also packet transmission. In an IEEE 802.15.4 network, the maximum frame size is limited, so additional bytes from encryption can influence fragmentation.

```text
[ Application Layer ]  Encrypted payload
[ Transport Layer   ]  UDP
[ Network Layer     ]  IPv6 + RPL
[ Adaptation Layer  ]  6LoWPAN
[ MAC/PHY Layer     ]  IEEE 802.15.4
```

### Why Fragmentation Matters

When the encrypted packet becomes too large, 6LoWPAN may divide it into multiple fragments. Each fragment requires extra headers and may increase the chance of retransmission. This means that energy consumption can increase even if the encryption computation itself is not the only cause.

The relationship can be summarized as follows:

```text
Larger encryption overhead
        ↓
Higher chance of 6LoWPAN fragmentation
        ↓
More transmitted packets
        ↓
Higher retransmission probability
        ↓
More energy consumption
```

This is why the project considered both algorithm-level performance and network-level overhead.

---

## 9. Trade-off Analysis

| Factor                         | ASCON                             | AES-GCM                      |
| ------------------------------ | --------------------------------- | ---------------------------- |
| Very small payloads            | Less efficient in our result      | More efficient in our result |
| Larger payloads                | More efficient in our result      | Energy increases faster      |
| Hardware compatibility         | May require proper 64-bit support | More widely supported        |
| Lightweight cryptography focus | Strong                            | Not designed mainly for LWC  |
| Standardization and deployment | Newer lightweight standard        | Very widely deployed         |
| Network overhead impact        | Lower growth trend                | Higher growth trend          |

The trade-off is not one-sided. AES-GCM is mature, widely used, and efficient for small payloads in our measurement. ASCON becomes more attractive when payload size increases and when the target platform properly supports its implementation.

---

## 10. Limitations

This experiment has several limitations.

First, the MSP430 environment caused problems when porting the actual ASCON code because of 64-bit operation constraints. Second, Cooja simulation does not perfectly represent real hardware energy measurement. Third, the 32-byte energy values were calibrated from additional Cooja timing runs rather than measured using a physical energy meter.

Therefore, the results should be interpreted as simulation-based evidence rather than final hardware-level proof.

---

## 11. Conclusion

This project compared ASCON and AES-GCM from the perspective of low-power IoT security. The results showed that AES-GCM was more energy-efficient for 16-byte payloads, while ASCON became more efficient as the payload size increased to 64 bytes.

The most important lesson from this project is that cryptographic algorithm selection should consider more than security strength alone. Payload size, hardware architecture, compiler support, packet fragmentation, and network retransmission can all affect the final energy cost.

For low-power IoT systems, ASCON is a strong candidate when payload sizes are moderate or large and the hardware platform supports it properly. AES-GCM remains useful when compatibility and small-payload efficiency are more important.

---

## 12. Future Work

Future experiments should include:

* Testing on real ARM Cortex-M hardware
* Measuring payload sizes larger than 128 bytes
* Comparing memory usage and code size
* Measuring DTLS handshake overhead
* Using physical energy measurement tools instead of simulation-only values

---

## 13. How to Run

### Prerequisites

```bash
git clone https://github.com/contiki-ng/contiki-ng
cd contiki-ng/tools/cooja
git clone https://github.com/contiki-ng/cooja.git .
```

### Run Cooja

```bash
docker run --privileged --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -e DISPLAY=host.docker.internal:0 -it --rm \
  -v C:/Users/{username}/contiki-ng:/home/user/contiki-ng \
  contiker/contiki-ng

cd contiki-ng/tools/cooja
sed -i 's/\r//' gradlew
./gradlew run
```

### Run the Simulation

1. Create a new simulation in Cooja.
2. Add UDP server and client nodes using the RPL UDP example.
3. Add ASCON and AES-GCM test nodes.
4. Start the simulation and check the mote output.
5. Record timing and energy-related values.

---

## 14. References

1. NIST, “Lightweight Cryptography Project,” 2023.
   https://csrc.nist.gov/projects/lightweight-cryptography

2. V. A. Thakor et al., “Lightweight Cryptography Algorithms for Resource-Constrained IoT Devices,” IEEE Access, 2021.

3. M. G. Spina et al., “An IoE-Powered Framework for Adaptive Energy-Security Trade-Off in IoT,” IEEE Network, 2026.

4. Contiki-NG Documentation.
   https://docs.contiki-ng.org/

5. ASCON Official Website.
   https://ascon.iaik.tugraz.at/
