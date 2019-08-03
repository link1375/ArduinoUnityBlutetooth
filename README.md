# ArduinoUnityBlutetooth

As I had trouble to use Sysem.IO.Ports in Unity, I created Unity3D plugin for Windows which uses python to communicate with Arduino. 

Disclaimer: This was the first time I used python and threading, so it is anything but perfect.

## Steps

1. Create a new Unity project and install python (3.7)
2. I inculuded the IronPython python plugin from exodrifter ([unity-python](https://github.com/exodrifter/unity-python)), created a new folder called `Plugins` and copied the content of Lib folder from the unity-python plugin into `Plugins/PythonLibs`. Most likely, there might be a better way to get those libs.
3. I downloaded the nuget packages [pythonnet](https://www.nuget.org/packages/Python.Runtime.NETStandard/) and [python37](https://www.nuget.org/packages/python), extracted them (7zip) and put the Python.Runtime.dll and python37.dll in the Plugin folder
4. In order to use the pySerial library I downloaded it from [pySerial](https://pypi.org/project/pyserial/) and copied the extracted pyserial-3.4 folder in the PythonLibs folder
5. I run the python code in a seperate task, since my framerate was constanly 1 fps on the main thread. I guess the call `var data = serialPort.read(4);` is blocking.
```
string filePath = Application.dataPath + "/Plugins/PythonLibs/pyserial-3.4";
CancellationToken ct = tokenSource.Token;

Task task = Task.Run(() =>
{
    PythonEngine.Initialize();
    try
    {
        using (Py.GIL())
        {
            dynamic sys = Py.Import("sys");
            sys.path.append(filePath);
            dynamic serial = Py.Import("serial");

            serialPort = serial.Serial(port);

            ct.ThrowIfCancellationRequested();

            while (true)
            {
                var data = serialPort.read(4);
                receivedData = data.decode("utf-8");
                Debug.Log(count + ": " + receivedData);
                ++count;

                if (ct.IsCancellationRequested)
                {
                    ct.ThrowIfCancellationRequested();
                    print("Cancel Task");
                }
            }
        }
    }
    finally
    {                
        serialPort.close();
        PythonEngine.Shutdown();
        Debug.Log("Finally");
    }
}, tokenSource.Token);
```

## Troubleshooting

1. Have you installed Python?
2. Set Api Compatability Level to .Net 4.x? `File > Build Settings > Player Settings > Other Settings`
3. Have you selected the right port? Before you can run the code, you have to connect the bluetooth device manually. Then open the Control Panel in Windows, go to `Hardware and Sound > Devices and Printers`. Double click on the bluetooth device, open the `Hardware`` tab and there you can see to which port your device is conneted.
