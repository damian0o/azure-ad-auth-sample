# Deploying FastAPI as a Windows Service with Uvicorn and Multiple Workers

## 1. Install Required Tools

Ensure the required libraries and tools are installed on your development system:

1. **Install Python dependencies**:
   ```bash
   pip install fastapi uvicorn pywin32 pyinstaller
   ```

2. **Install PyInstaller**:
   ```bash
   pip install pyinstaller
   ```

---

## 2. Write the FastAPI Application

Create and save your FastAPI application as `app.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI with Uvicorn workers!"}
```

---

## 3. Create the Service Wrapper

Create `service.py` to wrap the FastAPI application as a Windows service:

```python
import servicemanager
import win32serviceutil
import win32service
import win32event
import subprocess
import os
import sys


class FastAPIService(win32serviceutil.ServiceFramework):
    _svc_name_ = "FastAPIService"
    _svc_display_name_ = "FastAPI Application Service"
    _svc_description_ = "A FastAPI application running as a Windows service with Uvicorn workers."

    def __init__(self, args):
        super().__init__(args)
        self.hWaitStop = win32event.CreateEvent(None, 0, 0, None)
        self.process = None

    def SvcStop(self):
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        if self.process:
            self.process.terminate()
        win32event.SetEvent(self.hWaitStop)

    def SvcDoRun(self):
        app_path = os.path.abspath("app.py")  # Ensure this points to your app.py
        command = [
            sys.executable,
            "-m",
            "uvicorn",
            "app:app",
            "--host",
            "0.0.0.0",
            "--port",
            "8000",
            "--workers",
            "4"  # Specify the number of workers here
        ]
        self.process = subprocess.Popen(command, cwd=os.path.dirname(app_path))
        win32event.WaitForSingleObject(self.hWaitStop, win32event.INFINITE)


if __name__ == "__main__":
    win32serviceutil.HandleCommandLine(FastAPIService)
```

---

## 4. Package the Application into a Single Executable

Package the application and dependencies into a single `.exe` file:

```bash
pyinstaller --onefile --hidden-import=win32timezone service.py
```

- The `--onefile` option ensures the output is a single executable.
- The `--hidden-import=win32timezone` flag ensures compatibility with `pywin32`.

The executable will be generated in the `dist` directory as `service.exe`.

---

## 5. Deploy and Run the Service

1. **Copy the Executable**:
   Copy the `service.exe` file to the Windows Server.

2. **Install the Service**:
   Open a terminal or PowerShell as Administrator and run:

   ```bash
   service.exe install
   ```

3. **Start the Service**:
   Start the service with:

   ```bash
   service.exe start
   ```

Alternatively, use the Windows Service Manager (`services.msc`).

---

## 6. Configure Network Access

1. **Open the Port (8000)**:
   - Open Windows Defender Firewall.
   - Create an inbound rule to allow traffic on port 8000.

2. **Access the Application**:
   - From your internal network, access the app at:
     ```
     http://<server-ip>:8000
     ```

---

## 7. Verify the Service

- **Check Service Status**:
   ```bash
   service.exe status
   ```

- **Logs and Troubleshooting**:
   - Use Windows Event Viewer to inspect service logs for debugging.

---

## Notes

- **Workers**: You can adjust the `--workers` flag in `service.py` to suit your server’s resources.
- **Production Considerations**:
  - Test resource usage and adjust worker count for optimal performance.
  - Use load balancers if scaling beyond a single server.
