
# Build a Network Router from scratch

## Components of a router
1. PHY block
2. MAC block
3. Forwarding ASIC
4. Switch Fabric ASIC
5. CPU

## PHY block

PHY block implements a physical layer in OSI model. This PHY is short form of the physical layer, which implements a protocol to send to and receive from the
physical medium. It serves as physical connectivity between the medium which carries signal input and the digital circuitry.

PHY block has two major functions:
1. It has a analog domain acting as a transmitter and receiver of signals to/from the physical medium. This is Media Dependent Interface(MDI)
2. It interfaces with digital circuits like ASIC/CPU/FPGA/MCU providing Media Independent Interface (MII)

## MAC block

## Forwarding ASIC

## references:
1. https://e2e.ti.com/blogs_/b/analogwire/posts/simpliphy-your-ethernet-design-part-1-ethernet-phy-basics-and-the-selection-process
2. https://www.youtube.com/watch?v=o_2U1bk2SQM
3. 
