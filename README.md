# Mouse Eye Library.
Written by Joel Murphy, Fall 2010.
Updated by Joel Murphy, Spring 2018.
This Library offers full access to all the functionality
of the Avago ADNS2620 at the heart of the Mouse Eye.
Use of the Library is discribed here.

[ADNS2620 Datasheet](https://cdn.sparkfun.com/datasheets/Components/General%20IC/ADNS2620.pdf)

First, connect the SDIO and SCK lines from the Mouse Eye to the Arduino.
Any digital pins (2...13) will work for this connection.

To create an instance of the Mouse Eye:

	MouseEye yourName(int SCK_PIN, ing SDIO_PIN);

This library sets up 2 char variables and updates them through user functions.

	deltaX holds change in X since the last time you ran .getMouse()

	deltaY holds change in Y since the last time you ran .getMouse()


## USER FUNCTIONS

	getMouse();

This function updates the variables deltaX and deltaY when you call it.Variables will hold the change since the last time you called getMouse(). Calling getMouse() clears the X and Y registers. Movement distance result is encoded as a 2's compliment number. The char type resolves this.

---




	Cofig_Write(byte);

Writes a byte to the configuration register. Configuration register bits are:

Bit Position	| Function	|	Written 0 | Written 1
------------- | ------------- | ------------- | -------------
bit 7	| Reset	|	no effect |	reset the part
bit 6	| Power Down |	normal operation |  power down
bit 5	| Shutter Mode | Shutter Mode Off | Shutter Mode On
bit 4...1 |	RESERVED | NA | NA
bit 0 | Forced Awake | Normal | Always Awake

Shutter Mode Off means the LED is always on even if no movement up to 1 second
Shutter Mode On means LED only on when electronic shutter is open


	byte Config_Read();

Reads and returns the Configuration byte.


	byte Status();

Returns a byte that contains the product ID and mouse state data

	Bit Position	| Function	|	Value
------------- | ------------- | ------------- | -------------
bits 5- 7	| Product ID	|	010
bits 6 - 2	| RESERVED | NA 
bit 0 | Asleep/Awake | 0/1  

	int Squal();

Returns a value that reflects the surface quality of what the Mouse Eye sees.
This is the number of visible features on the surface.
It is useful to read this register if you are finding the ideal distance between you mouse and the surface it is looking at. Highest features = optimal distance.


	byte Max_Pixel();

Maximum gray scale value of a pixel in the current frame. Range = 0 to 63.


	byte Min_Pixel();

Minimum gray scale value of a pixel in the current frame. Range = 0 to 63.


	byte Pixel_Sum();

Returns the sum of all the pixels.


	word Shutter();

Length of time the shutter is open in clock cycles. The sensor adjusts shutter to keep average and maximum pixel values within normal operating range. Value may change. Each time it changes, it changes by +- 1/16 of current value.


	void yourName.F_Period_W(short);

Writes to the Frame_Period register. Default value is 194. For dark surfaces, use a lower number.
[see the datasheet for more details] (../assets/ADNS-2620.pdf)


	short F_Period_Read();

Reads the value in the Frame_Period Register.


	buyte Pixel_Data_Read();

Writes a byte to the `Pixel_Data` register to initiate frame capture and pixel dump. This command should be preceded by a write of 1 to the Config register set the mouse in Awake mode. Then there should be 324 successive reads from this register using Pixel_Data_Read(). Here is an example use of the Pixel_Data commands:


	MoueseEye Mouse(int SCK, int SDIO);	// create instance of MouseEye

	byte P_data [324];			// byte array to hold grayscale data

	void setup(){
	Serial.begin (9600);
	}

	void loop(){
		//some event triggers a call to dumpPixels();
	}

	void dumpPixels(){
	    Mouse.Config_W(1);             	// force awake
	    Mouse.Pixel_Data();		// writes a byte to initiate pixel dump
	    for (int i=0; i<324; i++){
	      P_data[i] = Mouse.Pixel_Data_R();	//read grayscale data
	    }
	    for (int i=0; i<324; i++){
	      Serial.print(P_data[i]); 		// display grayscale data
	    }
	    Mouse.Config_W(0);              	// undo force awake
	  }
