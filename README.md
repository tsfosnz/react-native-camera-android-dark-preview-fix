# react-native-camera-android-dark-preview-fix
A fix method for react-native-camera android dark preview issue

In my android device, when I use react-native-camera to open the preview, it appears more darker than system camera app. It's actually some code to initialize the camera before start preview.

This fix code is coming from https://android.googlesource.com/platform/packages/apps/Camera.git/+/lollipop-release

### Android

- RCTCamera.java

```java
    @SuppressWarnings("deprecation")
    public void updateCameraParametersInitialize(int cameraType) {

        Camera camera = this.acquireCameraInstance(cameraType);
        if (camera == null) {
            return;
        }

        // Reset preview frame rate to the maximum because it may be lowered by
        // video camera application.
        Camera.Parameters parameters = camera.getParameters();
        List<Integer> frameRates = parameters.getSupportedPreviewFrameRates();
        if (frameRates != null) {
            Integer max = Collections.max(frameRates);
            parameters.setPreviewFrameRate(max);
        }

        parameters.set("recording-hint", "false");

        // It's not necessary, as it could be translate from photo to video
        // Disable video stabilization. Convenience methods not available in API
        // level <= 14
        String vstabSupported = parameters.get("video-stabilization-supported");
        if ("true".equals(vstabSupported)) {
            parameters.set("video-stabilization", "false");
        }

        camera.setParameters(parameters);

    }
 ```
 
 In RCTCameraView.java:
 
 ```java
     public void setCameraType(final int type) {
        if (null != this._viewFinder) {
            this._viewFinder.setCameraType(type);
            RCTCamera.getInstance().adjustPreviewLayout(type);
        } else {
            _viewFinder = new RCTCameraViewFinder(_context, type);
            if (-1 != this._flashMode) {
                _viewFinder.setFlashMode(this._flashMode);
            }
            if (-1 != this._torchMode) {
                _viewFinder.setTorchMode(this._torchMode);
            }
            addView(_viewFinder);
        }
        /// fix is here
        RCTCamera.getInstance().updateCameraParametersInitialize(type);
        /// fix is here
    }
 
