## Why Docker?

Since this is the beginning of our Docker journey, it's worth understanding why we're using it.

Imagine you want to run these algorithms:

YOLOv11
ByteTrack
DeepSORT
MOTIP

Each may require different versions of Python or libraries. Installing everything directly on the TurboPi can lead to dependency conflicts.

Without Docker:

TurboPi
│
├── Python 3.9
├── OpenCV 4.5
├── YOLO
├── ByteTrack
├── DeepSORT
└── MOTIP

One package update can affect everything.

With Docker:

TurboPi
│
├── Container: YOLO
├── Container: ByteTrack
├── Container: DeepSORT
└── Container: MOTIP

Each container has its own isolated environment.

Each project has its own dependencies, making it easier to experiment and reproduce results.

Installation Plan

Once we've ensured there's enough storage, we'll proceed with:

Install Docker Engine.
Add the pi user to the docker group.
Verify the installation.
Install Docker Compose.
Run the classic hello-world container.
Create our first custom container.
Build a container for the TurboPi Dashboard backend.
Containerize the camera service.
Containerize YOLO and other algorithms.
