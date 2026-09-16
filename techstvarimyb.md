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