






































#include <pylon/PylonIncludes.h>
#include <pylon/PylonGUIIncludes.h>
#include <pylon/gige/BaslerGigEInstantCamera.h>
#include<pylon/InstantCameraArray.h>
typedef Pylon::CBaslerGigEInstantCamera Camera_t;
using namespace Basler_GigECameraParams;
using namespace Pylon;                                                                                                                    using namespace std;
static const size_t c_maxCamerasToUse = 2;
int main(int argc, char* argv[])
{
	// The exit code of the sample application.
	int exitCode = 0;
	int loopCount = 0;
	// Before using any pylon methods, the pylon runtime must be initialized. 

	Pylon::PylonAutoInitTerm autoInitTerm;
	try
	{
	       // Number of images to be grabbed.
		 static const uint32_t c_countOfImagesToGrab = 100;
		  // Get all attached devices and exit application if no device is found.
		 DeviceInfoList_t devices;
		CTlFactory& TlFactory = CTlFactory::GetInstance();
		CBaslerGigEDeviceInfo di, ai;
		di.SetSerialNumber("22349900");
		char const*SerialNumber2 = ("40009556");
		ai.SetSerialNumber("40009556");

		if (TlFactory.EnumerateDevices(devices) == 0)
		{
			throw RUNTIME_EXCEPTION("No camera present.");
		}
		// Create an array of instant cameras for the found devices and avoid exceeding a maximum number of devices.
		CInstantCameraArray Camera(min(devices.size(), c_maxCamerasToUse));
		// Create and attach all Pylon Devices.
		for (size_t i = 0; i < Camera.GetSize(); ++i)
		{
			Camera[i].Attach(TlFactory.CreateDevice(devices[i]));


			// Print the model name of the camera.
			cout << "Using device " << Camera[i].GetDeviceInfo().GetModelName() << endl;
		}

		// Starts grabbing for all cameras starting with index 0. The grabbing
	   // is started for one camera after the other. That's why the images of all
	   // cameras are not taken at the same time.
	   // However, a hardware trigger setup can be used to cause all cameras to grab images synchronously.
	   // According to their default configuration, the cameras are
	   // set up for free-running continuous acquisition.

		CInstantCamera Camera1(CTlFactory::GetInstance().CreateDevice(di));
		Camera1.StartGrabbing();
		// This smart pointer will receive the grab result data.
		CGrabResultPtr ptrGrabResult;
		// Grab c_countOfImagesToGrab from the cameras.
		for (uint32_t i = 0; i < c_countOfImagesToGrab && Camera1.IsGrabbing(); ++i)
		{
			Camera1.RetrieveResult(5000, ptrGrabResult, TimeoutHandling_ThrowException);

			// When the cameras in the array are created the camera context value
			// is set to the index of the camera in the array.
			// The camera context is a user settable value.
			// This value is attached to each grab result and can be used
			// to determine the camera that produced the grab result.
			intptr_t cameraContextValue = ptrGrabResult->GetCameraContext();
			//#ifdef PYLON_WIN_BUILD
						// Show the image acquired by each camera in the window related to each camera.
			Pylon::DisplayImage(cameraContextValue, ptrGrabResult);
			// Print the index and the model name of the camera.
			cout << "Cameras" << cameraContextValue << ": " << Camera1.GetDeviceInfo().GetModelName() << endl;
			cout << "SerialNumber : " << Camera1.GetDeviceInfo().GetSerialNumber() << endl;
			// Now, the image data can be processed.
			cout << "GrabSucceeded: " << ptrGrabResult->GrabSucceeded() << endl;
			cout << "SizeX: " << ptrGrabResult->GetWidth() << endl;
			cout << "SizeY: " << ptrGrabResult->GetHeight() << endl;
			const uint8_t *pImageBuffer = (uint8_t *)ptrGrabResult->GetBuffer();
			cout << "Gray value of first pixel: " << (uint32_t)pImageBuffer[0] << endl << endl;
		}
		Camera1.StopGrabbing();
		static const uint32_t c_loopCounterInitialValue = 10 * 4;
		loopCount = c_loopCounterInitialValue;
		while (loopCount > 0)
		{
			// Print a "." every few seconds to tell the user we're waiting for the callback.
			if (--loopCount % 4 == 0)
			{
				cout << ".";
				cout.flush();
			}
			WaitObject::Sleep(250);
		}
		CInstantCamera Camera2(CTlFactory::GetInstance().CreateDevice(ai));
		Camera2.StartGrabbing();
		for (uint32_t i = 0; i < c_countOfImagesToGrab && Camera2.IsGrabbing(); ++i)
		{
			Camera2.RetrieveResult(5000, ptrGrabResult, TimeoutHandling_ThrowException);
			intptr_t cameraContextValue = ptrGrabResult->GetCameraContext();
			//#ifdef PYLON_WIN_BUILD
			//Show the image acquired by each camera in the window related to each camera.
			Pylon::DisplayImage(cameraContextValue, ptrGrabResult);
			// Print the index and the model name of the camera.
			cout << "Cameras" << cameraContextValue << ": " << Camera2.GetDeviceInfo().GetModelName() << endl;
			cout << "SerialNumber : " << Camera2.GetDeviceInfo().GetSerialNumber() << endl;
			// Now, the image data can be processed.
			cout << "GrabSucceeded: " << ptrGrabResult->GrabSucceeded() << endl;
			cout << "SizeX: " << ptrGrabResult->GetWidth() << endl;
			cout << "SizeY: " << ptrGrabResult->GetHeight() << endl;
			const uint8_t *pImageBuffer = (uint8_t *)ptrGrabResult->GetBuffer();
			cout << "Gray value of first pixel: " << (uint32_t)pImageBuffer[0] << endl << endl;
		}
	}
	catch (const GenericException &e)
	{
		// Error handling
		cerr << "An exception occurred." << endl
			<< e.GetDescription() << endl;
		exitCode = 1;
	}
	// Comment the following two lines to disable waiting on exit.
	cerr << endl << "Press Enter to exit." << endl;
	while (cin.get() != '\n');
	// Releases all pylon resources. 
	return exitCode;
}





