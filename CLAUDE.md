# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a real-time camera processing application built with Kivy, OpenCV, and NumPy. The application captures video from a camera and applies various image processing filters in real-time.

## Development Commands

### Setup
```bash
pip install -r requirements.txt
```

### Running the Application
```bash
python main.py
# or
python3 main.py
```

### Testing Syntax
```bash
python3 -c "import ast; ast.parse(open('main.py').read()); print('Syntax OK')"
```

### Debugging
```bash
# Run with verbose output to see camera initialization
python3 main.py
```

## Architecture

The application follows a simple two-class architecture:

### CameraWidget (main.py:14-111)
- Extends Kivy's Image widget
- Manages OpenCV camera capture (cv2.VideoCapture)
- Handles real-time frame processing using Kivy's Clock scheduling
- Contains filter application logic with NumPy operations
- Converts OpenCV frames to Kivy textures for display
- Stores current frame for photo saving functionality
- Key methods:
  - `update_frame()`: Main processing loop scheduled at 30fps
  - `apply_filter()`: Processes frames with selected filter (main.py:53-87)
  - `start_camera()`/`stop_camera()`: Control camera lifecycle
  - `set_filter()`: Changes current filter (main.py:89-90)
  - `save_photo()`: Saves current frame to ~/Pictures with timestamp (main.py:92-106)
  - `release_camera()`: Cleanup camera resources (main.py:108-110)

### CameraApp (main.py:113-181)
- Main Kivy application class
- Creates GUI layout with camera display and control panel
- Uses BoxLayout for horizontal/vertical arrangement
- Binds UI controls to CameraWidget methods
- Handles application lifecycle (camera cleanup on exit)
- Filter definitions located in main.py:144-154

## Key Technical Details

### Frame Processing Pipeline
1. OpenCV captures BGR frames from camera
2. Filter applied via `apply_filter()` method
3. Current frame stored for photo saving (self.current_frame)
4. Frame converted BGR→RGB for Kivy compatibility
5. Frame flipped vertically (OpenCV/Kivy coordinate difference)
6. Frame converted to Kivy Texture and displayed

### Photo Saving
- Photos saved to ~/Pictures directory with timestamp format: `camera_photo_YYYYMMDD_HHMMSS.jpg`
- Captures the current filtered frame (not the original raw frame)
- Directory automatically created if it doesn't exist

### Filter Implementation
All filters implemented in `apply_filter()` method using:
- OpenCV functions (cv2.Canny, cv2.GaussianBlur, etc.)
- NumPy array operations for custom kernels
- Color space conversions (BGR, RGB, HSV, Grayscale)

### Dependencies
- kivy==2.2.0 (GUI framework)
- opencv-python==4.8.1.78 (camera capture and image processing)
- numpy==1.24.3 (array operations)
- pillow==10.0.1 (image support for Kivy)

## Camera Integration Notes

- Uses OpenCV VideoCapture(0) for default camera
- Requires camera permissions and exclusive access
- Frame rate set to 30fps via Kivy Clock scheduling
- Proper camera cleanup in app shutdown to prevent resource leaks

## Common Development Tasks

### Adding New Filters
1. Add filter logic to `apply_filter()` method in main.py:53-87
2. Add filter button entry to filters list in main.py:144-154
3. Follow existing filter patterns (check for self.current_filter == 'new_filter_name')

### Adding New UI Controls
- Add button creation in CameraApp.build() method after line 139
- Bind button to new method using `btn.bind(on_press=self.method_name)`
- Add corresponding method in CameraApp class that calls CameraWidget method
- Follow existing patterns like save_photo implementation

### Modifying GUI Layout
- Main layout structure defined in CameraApp.build() (main.py:114-160)
- Camera widget takes 70% width, controls take 30%
- All UI elements use BoxLayout with size_hint parameters

## Troubleshooting

### Camera Issues
- Check if camera is accessible: "Error: Could not open camera" printed to console
- Camera permissions may need to be granted
- Only one application can access camera at a time

### Performance Issues
- Frame rate controlled by self.fps (default 30fps) in main.py:16
- Heavy filters may cause frame drops - consider optimizing filter algorithms