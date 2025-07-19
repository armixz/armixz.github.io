---
layout: page
title: Operating System Process Simulator
description: Multi-process computer system simulation with IPC communication
img: assets/img/5.jpg
importance: 5
category: work
github: https://github.com/armixz/Computer-System-Simulation
---

## Project Overview

Developed a **multi-process computer system simulation** using C/C++ with separate CPU and Memory processes communicating via Inter-Process Communication in **Spring 2022**. This project demonstrates deep understanding of operating system concepts, including process scheduling, memory management, and low-level system programming with concurrent execution handling.

## System Architecture

### Process-Based Design
- **CPU Process**: Instruction fetch, decode, and execution simulation
- **Memory Process**: Memory allocation, deallocation, and management
- **Inter-Process Communication**: Message passing between CPU and Memory
- **Concurrent Execution**: Parallel process handling with synchronization

### Operating System Concepts
- **Process Scheduling**: Round-robin and priority-based scheduling algorithms
- **Memory Management**: Virtual memory simulation with paging
- **System Calls**: Simulated OS kernel interface
- **Process Control**: Fork, exec, wait, and signal handling

## Core Components

### CPU Simulator
- **Instruction Set Architecture**: Custom assembly-like instruction set
- **Execution Pipeline**: Fetch-decode-execute cycle implementation
- **Register Management**: CPU register file simulation
- **Program Counter**: Instruction pointer and flow control
- **Interrupt Handling**: Timer and I/O interrupt processing

### Memory Management System
- **Virtual Memory**: Address translation and page table management
- **Memory Allocation**: Dynamic memory allocation algorithms
- **Page Replacement**: LRU and FIFO page replacement policies
- **Memory Protection**: Segmentation and protection mechanisms
- **Cache Simulation**: L1/L2 cache hierarchy modeling

## Inter-Process Communication

### Message Passing Interface
```c
// IPC message structure
typedef struct {
    long msg_type;
    int operation;
    int address;
    int data;
    int process_id;
} ipc_message;

// Memory access request
int memory_read(int address) {
    ipc_message msg = {1, READ_OP, address, 0, getpid()};
    msgsnd(msgq_id, &msg, sizeof(msg) - sizeof(long), 0);
    
    msgrcv(msgq_id, &response, sizeof(response) - sizeof(long), getpid(), 0);
    return response.data;
}
```

### Synchronization Mechanisms
- **Message Queues**: System V IPC message queues for process communication
- **Shared Memory**: Memory segments for high-performance data sharing
- **Semaphores**: Process synchronization and mutual exclusion
- **Signals**: Asynchronous event notification between processes

## Process Scheduling

### Scheduling Algorithms
- **Round Robin**: Time quantum-based fair scheduling
- **Priority Scheduling**: Process priority-based execution ordering
- **Shortest Job First**: Execution time estimation and optimization
- **Multilevel Queue**: Multiple priority level management

### Process Control Block (PCB)
```c
typedef struct process_control_block {
    int pid;
    int priority;
    int state;  // READY, RUNNING, BLOCKED, TERMINATED
    int program_counter;
    int registers[NUM_REGISTERS];
    int memory_base;
    int memory_limit;
    int cpu_time_used;
    struct process_control_block* next;
} PCB;
```

## Memory Management Implementation

### Virtual Memory System
- **Address Translation**: Virtual to physical address mapping
- **Page Tables**: Multi-level page table implementation
- **TLB Simulation**: Translation Lookaside Buffer for fast address translation
- **Memory Mapping**: File and anonymous memory mapping

### Memory Allocation Strategies
- **First Fit**: First available block allocation
- **Best Fit**: Smallest suitable block selection
- **Worst Fit**: Largest available block allocation
- **Buddy System**: Power-of-2 block allocation algorithm

## System Features

### Process Management
- **Process Creation**: Fork-based process spawning
- **Process Termination**: Graceful and forced process cleanup
- **Process Communication**: Bidirectional message passing
- **Process Monitoring**: Real-time process state tracking

### Performance Monitoring
- **CPU Utilization**: Processor usage statistics
- **Memory Usage**: Physical and virtual memory consumption
- **Throughput Metrics**: Process completion rates
- **Response Time**: Interactive process response measurement

### Debugging and Visualization
- **Process State Display**: Real-time process status visualization
- **Memory Map**: Physical and virtual memory layout display
- **Execution Trace**: Instruction-by-instruction execution logging
- **Performance Graphs**: CPU and memory usage graphing

## Low-Level Programming Techniques

### System Programming
- **System Calls**: Direct kernel interface usage
- **Process Control**: Low-level process manipulation
- **Memory Mapping**: Direct memory access and manipulation
- **Signal Handling**: Asynchronous event processing

### Concurrent Programming
- **Race Condition Prevention**: Critical section protection
- **Deadlock Avoidance**: Resource allocation ordering
- **Thread Synchronization**: Mutex and condition variable usage
- **Atomic Operations**: Lock-free programming techniques

## Educational Value

This simulator provides hands-on experience with:
- **Operating System Design**: Core OS component implementation
- **System Programming**: Low-level C/C++ development
- **Process Management**: Real-world OS scheduling algorithms
- **Memory Systems**: Virtual memory and cache behavior
- **Concurrent Programming**: Multi-process synchronization

## Technologies Used

- **C/C++** for system-level programming
- **System V IPC** for inter-process communication
- **POSIX APIs** for process and thread management
- **Unix/Linux** system calls and interfaces
- **GDB** for debugging and development
- **Make** for build automation

The project demonstrates comprehensive understanding of operating system internals, system programming expertise, and low-level software development skills essential for systems engineering, embedded development, and performance-critical applications.
