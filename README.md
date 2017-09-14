Read the origin README [here](https://github.com/AaronShea/BLUI) 

Ignore all the stuff found in the original repo, i'll try to keep simple information here. 

## What does it do
BLUI "integrates" the [Chromium Embedded Framework (CEF)] (https://bitbucket.org/chromiumembedded/cef) into UE4.

It allows you to build stuff in HTML and show them into your application as a texture, but also show webpages and interact with them. You probably want to build your UI with it.

## Building
Again, just ignore the old references and the old building script. The best thing to do is to build the plugin starting from a precompiled CEF binary.
The original idea is to compile a mini browser (based on CEF's test simple browser) that can interpreter the blu_event() javascript function, and when it gets called it triggers an event that we can catch with the help of libcef_dll_wrapper (that we are going to build) in our UE4 app.

Just follows those steps:
  1. Download a "Standard Distribution" from [here] (http://opensource.spotify.com/cefbuilds/index.html). It includes precompiled binaries and the sources files for our libcef_dll_wrapper and the simple browser. Makes sure it matches your UE4 app build, which will probably be 64-bit. Extract it somewhere.
  2. Download cmake and install it.
  3. Clone the [browser repository](https://github.com/snaiperskaya96/BluBrowser.git) and extract the content of the BluBrowser folder into StandardCEFDistributionFolder/tests/cefsimple. Overwrite everything it asks you to overwrite.
  3. Navigate to where you unzipped the CEF distribution and run `cmake -G "Visual Studio 15 Win64" . -DUSE_SANDBOX=OFF` (Visual studio 15 is the alias for VS 2015 & 2017, adjust it accordingly with your VS version. See cmake docs) to generate the VS .sln files.
  4. Build the libcef_dll_wrapper using `cmake --build . --target libcef_dll_wrapper --config Release`
  5. Build the browser executable using `cmake --build . --target blu_ue4_process --config Release`
  6. Now clone this repository to YourProject/Plugins and navigate to YourProjects/Plugins/BLUI/ThirdParty (should be empty)
  7. Copy StandardCEFDistributionFolder/include to YourProjects/Plugins/BLUI/ThirdParty/include
  8. Copy StandardCEFDistributionFolder/Release/cef_sandbox.lib and libcef.lib to YourProject/Plugins/BLUI/ThirdParty/lib
  9. Copy StandardCEFDistributionFolder/libcef_dll_wrapper/Release/libcef_dll_wrapper.lib to YourProject/Plugins/BLUI/ThirdParty/lib
  10. Copy StandardCEFDistributionFolder/tests/cefsimple/Release to YourProject/Plugins/BLUI/ThirdParty/shipping.
  11. Regenerate your project VS files in the usual way.
  12. Hopefully it should compile smoothly
  
## Usage
Using the plugin is quite simple. At the end of the process you get a pointer to a Texture that show the current browser window. Apply it to whatever you want and you are golden. Below some steps to use BLUI with blueprint, same thing applies to C++, naming is very similar.

  1. Create a Material, create a TextureSampleParameter2D, set the Parameter Name to something and link it to the base color.
  2. From a blueprint (probably your umg) graph right click somewhere and look for the `Create Blueye` node.
  3. From the BluEye node get the `Set Properties` node and set it up as you prefer, you surely want to store the BluEye reference somewhere (see the output var from Set Properties). Put the TextureSampleParameter2D name in the Texture Parameter Name field.
  4. Drag the Init node from your BluEye object
  5. Now you can get the texture from the BluEye object using the "Get Texture" node and use it as you would
  6. Finally you have to trigger BluEye tick somewhere. I'd put it in the game level tick event, drag a node from it and search for `Run BLUI Tick`.