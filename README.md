# ChilkatXamarin

How to use chilkat library on Xamarin.Forms

1st: Open your Visual Studio for Mac and open your xamarin forms project you want to use Chilkat.
2nd: Create binding project to wrap native library

For iOS:

1. Download latest version of "Chilkat iOS Objective-C / Swift / C / C++" from https://www.chilkatsoft.com/downloads_ios.asp
	- Download and unzip to any directory
	- Chilkat provides static libs for each architecture slice: i386, x86_64 (for the simulator), armv7, armv7s, arm64, and armv6 (for older versions of iOS).
	- The Objective-C headers are contained in the "include" directory.
	- The C/C++ class headers are contained in the "cpp_include" directory.

2. Use libtool to "Building an iOS Universal Chilkat Static Library". One for Simulator and One for Device. Please don't mix them
	2.1 Cd to directory in step 1 and run 
		+ Build for Simulator: /usr/bin/libtool -static i386/libchilkatIos.a  x86_64/libchilkatIos.a -o libchilkatIos.a 
		+ Build for Device: /usr/bin/libtool -static armv7s/libchilkatIos.a  armv7/libchilkatIos.a arm64/libchilkatIos.a  -o libchilkatIos.a

3. Open your project on VS, click solution and create new project iOS >> Library >> Bindings Library (C#) 
	3.1 Create one binding project for Simulator and one binding project for Device
    3.2 All steps bellow you need to do two times for each projects.

4. Copy file "Static Library" from Step 2 "libchilkatIos.a" into binding project. Open VSMac and add exits file.
	4.1 VS detect and auto generate a file with the name "libchilkatIos.linkwith.cs"
	4.2 Open it and add new parameter, "LinkerFlag" with value "-lresolv -lpthread -lstdc++"
	==> [assembly: LinkWith ("libchilkatIos.a", SmartLink = true, ForceLoad = true, LinkerFlags = "-lresolv -lpthread -lstdc++")]

5. This step is really important
	- The binding project consists of the static library we just created and metadata in the form of C# code that explains how the Objective-C API can be used. This metadata is commonly referred to as the API definitions.
	- So in this our case, chilkat is a huge library, if you need use all functions of chilkat lib. Humzz, that fucking job :)) because you need define all functions into the ApiDefinition.cs. We have a tool "Objective Sharpie" can auto convert and generate all it for you, but you need review a result and fixing (Objective Sharpie does a great job of helping us, but it can't do everything.) 
	- So in my opinion, you just only define which feature you use
	- Let test in my case "Use Crypt feature"

6. Testing case --> Use Crypt/Decrypt feature
	6.1 Example on native (Object-C AES Encryption): https://www.example-code.com/objc/crypt2_aes.asp
		CkoCrypt2 *crypt = [[CkoCrypt2 alloc] init];
		crypt.CryptAlgorithm = @"aes";
		crypt.CipherMode = @"cbc";
		crypt.KeyLength = [NSNumber numberWithInt:256];
		crypt.PaddingScheme = [NSNumber numberWithInt:0];
		crypt.EncodingMode = @"hex";

		NSString *ivHex = @"000102030405060708090A0B0C0D0E0F";
		[crypt SetEncodedIV: ivHex encoding: @"hex"];

		NSString *keyHex = @"000102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F";
		[crypt SetEncodedKey: keyHex encoding: @"hex"];

		NSString *encStr = [crypt EncryptStringENC: @"The quick brown fox jumps over the lazy dog."];
		NSLog(@"%@",encStr);

		// Now decrypt:
		NSString *decStr = [crypt DecryptStringENC: encStr];
		NSLog(@"%@",decStr);

	6.2 Let reviewing example above, we need define in ApiDefinition.cs
		+ Class "CkoCrypt2" with properties "CryptAlgorithm,CipherMode,KeyLength,PaddingScheme,EncodingMode"
		+ Class "CkoCrypt2" with functions "SetEncodedIV,SetEncodedKey,EncryptStringENC,DecryptStringENC"

	6.3 Back to step 1, open directory of native lib, open "include" dir and open file "CkoCrypt2.h"	
		+ Find the fucking line define all properties and functions in section bellow(6.2)
		+ Look and make a define for each. See it on my source code


7. Add a reference from iOS project to binding project
	- Build and Use it :)


For Android: 
1. Download latest version of "Chilkat for Android™ Java" from https://www.chilkatsoft.com/chilkatAndroid.asp
	- Download and unzip to any directory
	- Chilkat provides libs for each architecture slice:  armeabi, mips, and mips64

2. In the directory Step 1, you can see the src folder with all the java files, we need convert it into a single jar libray file

2. Open your android project and create a new folder "jniLibs"
	- Copy the earch lib folder in Step 1 into this.
	- Set build action each file to "AndroidNativeLibrary"

3. Open MainAcitivy and add this line to load chilkat lib
	Java.Lang.JavaSystem.LoadLibrary("chilkat"); <-- Run on UIThread

4. Load lib is ok, but how to use???. Next step
