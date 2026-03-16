---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/init-py/"}
---


---

## Python `__init__.py`: The Package "Receptionist"

### 1. What is it?

The `__init__.py` file tells Python that a folder should be treated as a **Package** (a collection of modules). It acts as the **"Lobby"** or entry point for that folder.

### 2. The "Lobby" Shortcut

Instead of making users navigate deep into specific files, the developers use `__init__.py` to "expose" the most important tools at the folder level.

**Inside the `grid_integration` folder (`__init__.py`):**

The developers wrote these lines to pull tools from deep "offices" to the front "desk":

Python

```
from .tracker import Tracker
from .bidder import Bidder
from .coordinator import DoubleLoopCoordinator
```

### 3. Impact on your Plugin

Because of that file, your [thermal_generator_plugin.py](https://github.com/IDAES/idaes-pse/tree/3b76abca0acfba95a5bbaaf77fcd97690a3b0b96/idaes/apps) can use a much cleaner import style:

- **The Professional Way (using the "Lobby"):**
    
    `from idaes.apps.grid_integration import DoubleLoopCoordinator`
    
- **The "Messy" Way (skipping the receptionist):**
    
    `from idaes.apps.grid_integration.coordinator import DoubleLoopCoordinator`
    

### 4. Why this is useful

- **Cleaner Code:** Your plugin file stays short and readable.
    
- **Abstraction:** If the IDAES team moves the `DoubleLoopCoordinator` code to a different file (like `manager.py`), your plugin won't break—as long as they update the `__init__.py` "receptionist," your code remains exactly the same.
    
- **Organization:** It turns a bunch of separate `.py` files into one unified **Toolkit**.
    

---

