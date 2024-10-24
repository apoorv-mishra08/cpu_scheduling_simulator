# CPU Scheduling Simulator

This project is a CPU scheduling simulator built using Python and Streamlit. It supports the following CPU scheduling algorithms:
- First Come First Serve (FCFS)
- Round Robin (RR)
- Shortest Job First (SJF)

## How to Run the Project

1. Clone the repository:
   ```bash
   git clone https://github.com/apoorv-mishra08/cpu_scheduling_simulator.git
**Install the necessary Python packages:**

pip install -r requirements.txt
Run the Streamlit app:

streamlit run cpu_scheduling_simulator.py

Final Code:

import streamlit as st
import matplotlib.pyplot as plt

# First Come First Serve (FCFS) Scheduling
def fcfs_scheduling(processes):
    processes.sort(key=lambda x: x[1])  # Sort by arrival time
    time = 0
    start_times = []
    durations = []
    
    for p in processes:
        if time < p[1]:  # If the current time is less than the process arrival time, wait
            time = p[1]
        start_times.append(time)
        durations.append(p[2])  # Burst time
        time += p[2]  # Increment time by burst time
    
    return start_times, durations

# Round Robin Scheduling
def round_robin_scheduling(processes, quantum):
    queue = []
    time = 0
    start_times = [-1] * len(processes)  # Start times for each process
    remaining_burst_times = [p[2] for p in processes]  # Remaining burst time for each process
    durations = [0] * len(processes)  # Total duration each process runs
    timeline = []  # To keep track of process execution at each time slice

    while processes or queue:
        # Add processes that have arrived by current time
        while processes and processes[0][1] <= time:
            queue.append(processes.pop(0))
        
        if queue:
            p = queue.pop(0)
            process_index = p[0] - 1  # process ID - 1 for index
            
            if start_times[process_index] == -1:
                start_times[process_index] = time  # Set start time when process first starts
            
            if remaining_burst_times[process_index] > quantum:
                timeline.append((time, quantum, p[0]))  # Add to timeline
                time += quantum
                remaining_burst_times[process_index] -= quantum
                queue.append(p)  # Process is not done, so it goes back in the queue
            else:
                timeline.append((time, remaining_burst_times[process_index], p[0]))  # Add to timeline
                time += remaining_burst_times[process_index]  # Process finishes
                remaining_burst_times[process_index] = 0  # Mark process as completed
                durations[process_index] = time - start_times[process_index]  # Total duration
        else:
            time += 1  # If no process is available, increment time

    return start_times, durations, timeline

# Shortest Job First (SJF) Scheduling
def sjf_scheduling(processes):
    processes.sort(key=lambda x: (x[1], x[2]))  # Sort by arrival time, then by burst time
    time = 0
    start_times = []
    durations = []
    timeline = []  # To keep track of process execution

    while processes:
        available_processes = [p for p in processes if p[1] <= time]
        if available_processes:
            p = min(available_processes, key=lambda x: x[2])  # Process with shortest burst time
            processes.remove(p)
            start_times.append(time)
            durations.append(p[2])  # Burst time
            timeline.append((time, p[2], p[0]))  # Track timeline for Gantt chart
            time += p[2]  # Increment time by burst time
        else:
            time += 1  # If no process is ready, increment time by 1
    
    return start_times, durations, timeline

# Streamlit app
st.title("CPU Scheduling Simulator")

# Input number of processes
num_processes = st.number_input("Enter number of processes", min_value=1, step=1)

processes = []
for i in range(num_processes):
    arrival_time = st.number_input(f"Enter arrival time for process {i+1}", min_value=0, step=1)
    burst_time = st.number_input(f"Enter burst time for process {i+1}", min_value=1, step=1)
    processes.append((i+1, arrival_time, burst_time))

# Select scheduling algorithm
algorithm = st.selectbox("Select Scheduling Algorithm", ["FCFS", "Round Robin", "Shortest Job First"])

# Additional input for Round Robin
quantum = None
if algorithm == "Round Robin":
    quantum = st.number_input("Enter quantum time", min_value=1, step=1)

if st.button("Run Scheduling"):
    if algorithm == "FCFS":
        start_times, durations = fcfs_scheduling(processes)
        timeline = [(start_times[i], durations[i], i+1) for i in range(len(processes))]  # For Gantt chart
    elif algorithm == "Round Robin":
        if quantum is not None:
            start_times, durations, timeline = round_robin_scheduling(processes, quantum)
        else:
            st.error("Please provide a quantum value for Round Robin scheduling.")
    elif algorithm == "Shortest Job First":
        start_times, durations, timeline = sjf_scheduling(processes)
    
    st.write("Start Times:", start_times)
    st.write("Durations:", durations)

    # Plotting the Gantt chart
    fig, ax = plt.subplots()
    for start_time, duration, process in timeline:
        ax.broken_barh([(start_time, duration)], (10 * (process - 1), 9), facecolors=('tab:blue'))
    
    ax.set_ylim(0, 10 * len(processes))
    ax.set_xlim(0, max(start_times) + max(durations))
    ax.set_xlabel('Time')
    ax.set_ylabel('Processes')
    ax.set_yticks([10 * i + 5 for i in range(len(processes))])
    ax.set_yticklabels([f'P{i+1}' for i in range(len(processes))])
    ax.grid(True)
    
    st.pyplot(fig)

