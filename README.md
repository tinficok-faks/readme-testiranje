# Post-Enrolment Course Timetabling (PECT)

A university course timetabling application project developed for solving the **Post-Enrolment Course Timetabling (PECT)** problem. The project combines a Python/CustomTkinter GUI with C++ implementations of constructive and local-search based approaches.

> **Important:** Before running the application, install the GUI dependency:
>
> ```bash
> pip install customtkinter
> ```
>
> The project also requires a working **C++ compiler** and **Make** (`make` or `mingw32-make`), because the GUI builds the C++ solvers when an algorithm is started.

---

## Table of Contents

- [Problem Overview](#-problem-overview)
- [Features](#-features)
- [Algorithms](#-algorithms)
- [Getting Started](#-getting-started)
- [How to Use the GUI](#-how-to-use-the-gui)
- [Output and Evaluation](#-output-and-evaluation)
- [Project Structure](#-project-structure)
- [Screenshots](#-screenshots)
- [Datasets](#-datasets)
- [Technical Notes](#-technical-notes)

---

## About the problem

The **Post-Enrolment Course Timetabling** problem schedules university events after students have already selected the courses they want to attend. The timetable assigns each event a **timeslot** and a **room**, while respecting hard constraints and trying to minimize soft-constraint violations. This formulation was used as Track 2 of the Second International Timetabling Competition (ITC2007). The official problem description emphasizes that knowing student enrolments at timetable-construction time allows the timetable to be built around actual student choices.

In this project, each instance contains events, rooms, room features, students, event availability, student attendance information, and precedence requirements. The implementation works with the competition-style `.tim` datasets and 45 timeslots (5 working days × 9 periods), matching the assignment specification.


### Hard constraints

A valid timetable must respect the following constraints:

- Students cannot attend two events at the same time.
- An assigned room must have sufficient capacity and all required features.
- Two events cannot occupy the same room at the same timeslot.
- Events can only be scheduled in available timeslots.
- Precedence requirements between events must be respected where specified.

### Soft constraints

The project also evaluates undesirable timetable patterns:

- a student having an event in the last timeslot of a day
- three or more consecutive events in one day
- having only one event in a day

A solution may leave some events unplaced and still remain **valid**. The resulting **distance to feasibility** is the number of students who would attend the unplaced events. A solution with no unplaced events and no violated hard constraints is **feasible**.

### More about the problem and said competition:
- [ITC2007 – International Timetabling Competition](https://www.eeecs.qub.ac.uk/itc2007/index.htm)
- [ITC2007 – Post Enrolment Course Timetabling](https://www.eeecs.qub.ac.uk/itc2007/postenrolcourse/course_post_index.htm)
- [Official PECT problem description](https://www.eeecs.qub.ac.uk/itc2007/postenrolcourse/report/Post%20Enrolment%20based%20CourseTimetabling.pdf)

---

## Features

- CustomTkinter desktop GUI for selecting datasets and running algorithms
- Support for all 24 listed datasets (`dataset1.tim` ... `dataset24.tim`)
- Greedy timetable construction.
- Tabu Search used as a starting solution for local search.
- Four local-search configurations:
  - Best-improving Transfer
  - First-improving Transfer
  - Best-improving Swap
  - First-improving Swap
- Automatic timetable validation using `check.cpp` / `check.exe`
- Automatic calculation and display of hard-constraint and soft-constraint costs
- Room-by-room timetable visualization directly inside the GUI
- Automatic generation of solution files in the corresponding output directories

---

## Algorithms

### Greedy Method

The greedy solver parses the selected `.tim` instance, builds a conflict graph, constructs a timetable, validates it, and attempts to repair it when necessary. The resulting schedule is then evaluated and written to the greedy output directory.

### Tabu Search

Tabu Search is used to produce a common starting solution for the local-search methods. The current implementation solves the selected dataset and stores the resulting schedule in `tabu_outputs`.

### Local Search

Two neighbourhood concepts are implemented:

**Transfer neighbourhood** — moves an event from its current assignment to another valid assignment.

**Swap neighbourhood** — exchanges assignments between events.

For both neighbourhoods the implementation provides:

- **Best-improving neighbour** — examines candidates and chooses the best improvement.
- **First-improving neighbour** — stops at the first improving candidate.

The executable accepts method numbers `1–4` for these combinations, while `-1` generates only the Tabu Search solution.

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/tinficok-faks/Post-Enrolment-Course-Timetabling.git
cd Post-Enrolment-Course-Timetabling
```

### 2. Install the Python GUI dependency

```bash
pip install customtkinter
```

### 3. Make sure the build tools are installed

The GUI searches for either `make` or `mingw32-make` and uses it to build the C++ projects. A working `g++` installation is also required because the validator is compiled automatically when the greedy method is started.

### 4. Start the application

```bash
python app.py
```

The application creates a window titled **PECT - Generator rasporeda** and provides controls for dataset selection, algorithm execution, cleaning generated files, and timetable/result display.

---

## 🖥️ How to Use the GUI

### Step 1 — Select a dataset

Click **Select Dataset** and choose one of the available `.tim` files from the `datasets/` directory.

The current GUI expects files named `dataset1.tim` through `dataset24.tim`.

### Step 2 — Run an algorithm

Choose one of the available methods:

| GUI action | Description |
|---|---|
| **Greedy Method** | Constructs a timetable using the greedy approach. |
| **Best-improving Transfer** | Local search with Transfer neighbourhood. |
| **First-improving Transfer** | Local search with Transfer neighbourhood. |
| **Best-improving Swap** | Local search with Swap neighbourhood. |
| **First-improving Swap** | Local search with Swap neighbourhood. |

For local search, the application first prepares a Tabu Search solution and then uses it as the starting schedule.

### Step 3 — Inspect the result

After execution, the GUI displays:

- the selected algorithm and dataset;
- the number of unplaced events;
- the validation result;
- a room-by-room timetable covering the 45 available timeslots.

Each room is shown as a separate card containing its capacity and a compact Monday–Friday timetable.

---

## 📊 Output and Evaluation

The assignment requires the generated results to report:

| Value | Meaning |
|---|---|
| **Distance to Feasibility** | Total number of students associated with unplaced events. |
| **Soft Cost** | Number of violated soft constraints. |
| **Total Cost** | Distance to Feasibility + Soft Cost. |

For the project, a schedule with hard-constraint violations is treated separately from a valid timetable, while feasible solutions have distance to feasibility equal to `0`.

The validator reads the `.tim` input, checks room capacity/features, availability, precedence, student conflicts and room conflicts, and then evaluates the soft constraints.

The GUI interprets the validator return value as follows:

- positive value → soft-constraint cost;
- negative value → hard-constraint violations;
- zero → feasible timetable with no reported soft cost.

---

## 📂 Project Structure

The following structure follows the organization used by the project and is intentionally presented in a compact, documentation-oriented form.

```text
Post-Enrolment-Course-Timetabling/
├── datasets/                         # ITC-style .tim problem instances
│   ├── dataset1.tim
│   ├── dataset2.tim
│   └── ...
│
├── greedy/                           # Greedy solver
│   ├── main.cpp                      # Solver entry point
│   ├── graph.*                       # Conflict graph representation
│   ├── parser.*                      # .tim instance parser
│   ├── timetable.*                   # Timetable construction/evaluation helpers
│   └── Makefile                      # Build rules
│
├── local_search/                     # Tabu Search + Local Search
│   ├── main.cpp                      # Algorithm entry point
│   ├── filereader.*                  # Input/solution loading
│   ├── tabu_search.*                 # Tabu Search implementation
│   ├── local_search.*                # Transfer and Swap neighbourhoods
│   ├── output.*                      # Solution writing helpers
│   └── Makefile                      # Build rules
│
├── greedy_outputs/                   # Greedy-generated solution files
├── tabu_outputs/                     # Tabu Search solution files
├── ls_outputs/
│   ├── ls1_outputs/                  # Best-improving results
│   └── ls2_outputs/                  # First-improving results
│
├── app.py                            # CustomTkinter GUI
├── check.cpp                         # Timetable validator
├── check.exe                         # Validator executable (generated/used on Windows)
└── README.md                         # Project documentation
```

The top-level directories used by the GUI are defined explicitly in `app.py`, including `datasets`, `greedy`, `local_search`, `greedy_outputs`, `tabu_outputs`, `ls_outputs/ls1_outputs`, and `ls_outputs/ls2_outputs`.

The C++ entry points confirm the separation between the greedy solver and the local-search/Tabu Search implementation.

---

## 🖼️ Screenshots

The project is especially suitable for showing the GUI directly in the README. Recommended screenshots are:

### Main GUI

Show the application immediately after launch, before a dataset is selected. This should clearly show the header, dataset selection area, algorithm buttons and the empty result section.

```md
![Main GUI](screenshots/main_gui.png)
```

### Dataset Selected

Show the GUI after selecting an instance such as `dataset1.tim`, with the selected dataset visible and the algorithm controls ready.

```md
![Dataset selected](screenshots/dataset_selected.png)
```

### Generated Timetable

Show the results panel after an algorithm finishes. The most useful view contains the algorithm name, validation/cost information, unplaced-event information and several room cards with the Monday–Friday timetable.

```md
![Generated timetable](screenshots/generated_timetable.png)
```

> **Screenshot note:** I was not able to execute the complete repository in this environment because the repository could not be cloned into the execution environment and the project depends on the full C++ source tree plus local build tools. The GUI layout and the recommended screenshots above are therefore based on the actual `app.py` implementation rather than fabricated runtime images. The GUI code confirms a 1480×900 default window, dataset-selection controls, five algorithm buttons and room-by-room result cards.

---

## 📁 Datasets

The assignment requires the available instances in the `datasets` directory to be used. The GUI currently accepts dataset numbers from **1 to 24** and expects the corresponding filenames `datasetN.tim`.

The official ITC2007 Post-Enrolment Course Timetabling page provides the problem model, instance information, input format, output format and validation resources.

---

## 🔧 Technical Notes

### GUI ↔ C++ integration

The Python GUI does not implement the optimization algorithms itself. Instead, it:

1. selects the input dataset;
2. builds the relevant C++ project with `make` or `mingw32-make`;
3. runs the generated executable;
4. reads the produced `.sln` solution file;
5. validates the schedule with `check.exe`;
6. parses the schedule and renders it as room cards in the GUI.

### Solution format

The GUI expects each event to be represented by a line containing:

```text
<timeslot> <room>
```

An unplaced event is represented as:

```text
-1 -1
```

The GUI converts the solution into `room → timeslot → event` mappings for visualization.

### Timeslots

The implementation uses **45 timeslots**, organized as **5 days × 9 slots per day**. The final five slots of each day are represented by the indices used for the soft constraint concerning end-of-day events.

---
