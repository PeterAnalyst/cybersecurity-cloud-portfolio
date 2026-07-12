# Project 1: Home Lab Setup
**Date completed:** July 12 2026
**Duration:** Approximately 1.5 hours

## What I Built
A professional cybersecurity home lab
using VirtualBox and Kali Linux running
on a Windows host machine.

## Environment Specifications

### Host Machine
- OS: Windows
- RAM: 16GB
- Storage: 476GB SSD

### Virtual Machine: Kali Linux 2026.1
- RAM allocated: 4096MB
- CPU allocated: 2 cores
- Video Memory: 128MB
- Network mode: NAT
- Storage: 80.09GB virtual disk

## Configuration Steps Completed
1. Confirmed VirtualBox 7.2.10 installed
2. Imported Kali Linux 2026.1 VirtualBox image
3. Configured VM settings (RAM, CPU, display, network)
4. Changed default credentials
5. Updated system (2,962MB updates applied)
6. Verified internet connectivity
7. Verified security tools operational
8. Created recovery snapshot

## Verification Tests

### Test 1: Internet Connectivity
Command: ping google.com
Result: 13/13 packets received, 0% packet loss
Average latency: 4.515ms
Status: PASSED

### Test 2: Security Tools
Command: nmap --version
Result: Nmap 7.98 confirmed installed
Status: PASSED

### Test 3: Network Configuration
Command: ip addr show
Result: eth0 UP, IP 10.0.2.15 (NAT)
MAC Address: 08:00:27:8a:35:d2
Status: PASSED

## Snapshot
Name: Clean Kali 2026.1 Post Update July 2026
Purpose: Recovery point if lab environment
         becomes corrupted during exercises

## Skills Demonstrated
- Virtualisation setup and configuration
- Linux first boot and initial configuration
- Linux package management (apt update/upgrade)
- Linux terminal commands (passwd, ping, nmap, ip)
- Professional lab documentation
- Snapshot management for disaster recovery

## Next Project
Project 2: Network Traffic Analysis with Wireshark
Scheduled: Day 28 of curriculum
