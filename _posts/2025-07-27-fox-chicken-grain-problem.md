---
layout: post
title: "fox, chicken and grain problem"
date: 2025-07-27
---

### Problem

The classic "fox, chicken, and grain" river crossing puzzle:

- The farmer wants to take "fox, chicken, and grain" 3 items crossing river

- With boat, the farmer can each time take at most 1 item

- On the riverbank, if the farmer is absent, fox will eat chicken and chicken will eat grain. If the farmer is present, such eating situation won't happen.

Please print the steps how the farmer is crossing the river.

### ChatGPT answer

It's a fantastic state-space search problem, often solved by **BFS** (breadth-first search) or **DFS** with state checking to avoid cycles.

- Each riverbank’s status is tracked (`true` = left bank, `false` = right bank).

- State is encoded for both left and right riverbank

- For each move, the code checks if any “eating” scenario happens (invalid state).

**Output** is below

```
Step 0: Start
Step 1: Farmer takes chicken left->right
Step 2: Farmer takes nothing right->left
Step 3: Farmer takes grain left->right
Step 4: Farmer takes chicken right->left
Step 5: Farmer takes fox left->right
Step 6: Farmer takes nothing right->left
Step 7: Farmer takes chicken left->right
```

Code **example**

```java
import java.util.*;

public class Main {

  static class State {
    int[] left; // left bank [farmer, grain, chicken, fox]
    int[] right; // right bank [farmer, grain, chicken, fox]
    List<String> steps;

    State(int[] left, int[] right, List<String> steps) {
      this.left = Arrays.copyOf(left, 4);
      this.right = Arrays.copyOf(right, 4);
      this.steps = new ArrayList<>(steps);
    }

    String encode() { // for visited state check
      return Arrays.toString(left) + Arrays.toString(right);
    }
  }

  static boolean isValid(int[] side) {
    // 0:farmer, 1:grain, 2:chicken, 3:fox
    if (side[0] != side[2]) {
      if (side[1] == side[2]) return false; // grain & chicken together, no farmer
      if (side[2] == side[3]) return false; // chicken & fox together, no farmer
    }
    return true;
  }

  static void solve() {
    int[] startLeft = {1, 1, 1, 1};
    int[] startRight = {0, 0, 0, 0};

    Queue<State> queue = new LinkedList<>();
    Set<String> visited = new HashSet<>();
    queue.offer(new State(startLeft, startRight, List.of("Start")));

    while (!queue.isEmpty()) {
      State s = queue.poll();

      // Goal: all on right
      if (Arrays.equals(s.right, new int[] {1, 1, 1, 1})) {
        int index = 0;
        for (String step : s.steps) System.out.printf("Step %s: %s%n", index++, step);
        return;
      }

      String code = s.encode();
      if (visited.contains(code)) continue;
      visited.add(code);

      // Farmer on left
      if (s.left[0] == 1) {
        for (int carry = 0; carry <= 3; carry++) {
          if (carry == 0 || s.left[carry] == 1) { // carry nothing or carry one thing
            int[] newLeft = Arrays.copyOf(s.left, 4);
            int[] newRight = Arrays.copyOf(s.right, 4);
            newLeft[0] = 0;
            newRight[0] = 1; // farmer moves right
            if (carry > 0) {
              newLeft[carry] = 0;
              newRight[carry] = 1;
            }
            if (isValid(newLeft) && isValid(newRight)) {
              List<String> newSteps = new ArrayList<>(s.steps);
              String move =
                  "Farmer takes "
                      + (carry == 0
                          ? "nothing"
                          : carry == 1 ? "grain" : carry == 2 ? "chicken" : "fox")
                      + " left->right";
              newSteps.add(move);
              queue.offer(new State(newLeft, newRight, newSteps));
            }
          }
        }
      }

      // Farmer on right
      if (s.right[0] == 1) {
        for (int carry = 0; carry <= 3; carry++) {
          if (carry == 0 || s.right[carry] == 1) {
            int[] newLeft = Arrays.copyOf(s.left, 4);
            int[] newRight = Arrays.copyOf(s.right, 4);
            newLeft[0] = 1;
            newRight[0] = 0; // farmer moves left
            if (carry > 0) {
              newLeft[carry] = 1;
              newRight[carry] = 0;
            }
            if (isValid(newLeft) && isValid(newRight)) {
              List<String> newSteps = new ArrayList<>(s.steps);
              String move =
                  "Farmer takes "
                      + (carry == 0
                          ? "nothing"
                          : carry == 1 ? "grain" : carry == 2 ? "chicken" : "fox")
                      + " right->left";
              newSteps.add(move);
              queue.offer(new State(newLeft, newRight, newSteps));
            }
          }
        }
      }
    }
    System.out.println("No solution found!");
  }

  public static void main(String[] args) {
    solve();
  }
}

```
