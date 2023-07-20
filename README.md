# OSCAD-Zento
## Case builder for openSCAD

Here I present a construction framework for openSCAD.  
I describe the concept using the example of a case for microcontrollers with display and some keys.  
The finished case can be printed with a 3D printer.
## Part 1
#### Everything is a module
he single parts like housing, microcontroller, LCD, buttons etc.
* must be able to be made visible or invisible depending on the 3D printing. 
* be able to be positioned and aligned at any place. 

For all modules I use a uniform parameter structure:
```
//               View  Position  Parameter  $fn
module ArduNanoE(V,    Ptr,      M,        fn  ) {         
}
```

Each module must have at least 2 parameters
1. View: Value 0 to... any, 0: the part is not visible, >0 visible
2. Position: Array[px,py,pz, rx,ry,rz] for position translate([x,y,z])  rotate([x,y,z])  

Additionally further parameters can be passed
e.g. M for module specific parameters, there can be more than one. The last parameter should be $fn for cylinder for 3D print quality.

#### Example  
```
// simple example
// ============== Control Center ==========================
Vc=1;               // View 0..4 Cube
Vy=1;               // View 0..2 Cylinder
//     tx ty tz  ry ry rz
Ptrc=[ 0, 0, 0,  0, 0, 0]; // Position translate rotate cube
Ptry=[-1,10, 7,  0,90, 0]; // Position translate rotate cylinder
Cd=[4,70,20];       // Cube     dimension x,y,z
Cy=[6,8];           // Cylinder dimension h,d
fn=24;              // $fn quality  cylinder
CU=[Vc,Ptrc,Cd];    // array module Cube
CY=[Vy,Ptry,Cy,fn]; // array module Cyli1

//============== Show Module =========================
//difference(){ //     View,Position,Parameter,$fn
  color("lime")  Cube(CU[0],CU[1],   CU[2]          );
  color("coral") Cyl1(CY[0],CY[1],   CY[2],    CY[3]);
// }

//============ Working Module============
// you can put this modules in a Library for use
module Cube(V, Ptr,   C) {
  if (V>0) { 
    translate([Ptr[0],Ptr[1],Ptr[2] ]) rotate([ Ptr[3],Ptr[4],Ptr[5] ]) 
      cube(C); 
} }
// 
module Cyl1(V, Ptr,   C,   fn) {
  if (V>0) {  
    translate([Ptr[0],Ptr[1],Ptr[2]]) rotate([Ptr[3],Ptr[4],Ptr[5]]) { 
      if (V==1) cylinder(h=C[0],d=C[1],center=false,$fn=fn);
      if (V==2) cylinder(h=C[0],d=C[1],center=true, $fn=fn); 
} } } }
```

#### Clipping
To position a e.g. hole in a wall of a housing, one needs e.g. a cube and a cylinder.  
Here is a small example with 2 modules: a cylinder and a cube module.  

The file CubeCyli.scad contains the two modules.  
The two modules and their associated parameters are inserted into a differnce().  
The array V contains the on/off switches of the modules.  

```
//  SimpleExample.scad 
use <CubeCyli.scad>;

// View settings 	possible default
//  cube         	0..1     1 
//  ↓  cylinder  	0..2     1 
//  ↓  ↓ 
 V=[1, 1]; // on/off for cube, cylinder 

fn=24;     // 3D quality cylinder, 3d-Print: 30..60

Cu=[4,70,20];            // dimension cube
Cy=[5,8];                // dimension cylinder
Ptr01=[0,0,0,  0, 0,0];  // position rotation cube
Ptr02=[0,12,8, 0,90,0];  // position rotation cylinder 

//   View  Position  Parameetr $fn   
A01=[V[0], Ptr01,    Cu          ];
A02=[V[1], Ptr02,    Cy,       fn]; 

difference() {
//        View    Position  Parameter  $fn
    Cube1(A01[0], A01[1],   A01[2]); 
// clipping
  #Cyl1(A02[0],  A02[1],  A02[2],  A02[3]); 
} // ---that's all---
```

To simply make a hole in a plate is of course total overkill. This is only to illustrate the basic construction. 

## Part 2
#### The complete NanoHouse example
The example includes the 4 include files with the modules to be called.   
A special feature here is === VIEV AND 3D-SETTINGS===   
By this array e.g. V=[14,7,7,1,3,1]; different views can be stored for 3D printing.   
Only one array may be active at a time, comment out the others accordingly! 

In the difference there are also special clipping modules.  
There are always 2 modules belonging together that access the same parameter set.  

OLED13(..); und OLED13C(..);  
Buttons3(..); und  Buttons3C(..);  
ArduNanoE(..); und ArduNanoEC(..);  

In connection with the View switch all superfluous display details can be hidden!
  
```
// NanoHouse with Arduino Nano, 128x64 OLED, 3 Buttons
//==================================================================
//------constuction compleatly with parameters for 3D-prints--------

include <0_MicroControler.scad>; // Arduino Uno, Mini, Nano ...
include <0_House.scad>;          // module for building housing
include <0_LCD-OLED-LED.scad>;   // 2x16, 4x20, 64x128, oled
include <0_ButtonSwitch.scad>;   // buttons, rotary, switches

/*
=== VIEW AND 3D-PRINT SETTINGS =====================================
                      settings: possible 3dprint 
      House Panels              0..45     23    test: 14 24 34 44
      ↓ Sunk Screws             0..7       7
      ↓ ↓ Beams                 0..7       7
      ↓ ↓ ↓ OLED                0..3       3
      ↓ ↓ ↓ ↓ Buttons           0..4       3
      ↓ ↓ ↓ ↓ ↓ ArduinoNano     0..4       3
      ↓ ↓ ↓ ↓ ↓ ↓
 No.[ 0,1,2,3,4,5] */
//V=[14,7,7,1,3,1];  // view  test, debug
  V=[24,7,0,3,3,0];  // 3D-print Panel top   topical view!
//V=[23,0,7,0,0,3];  // 3D-print house down
//V=[ 0,0,0,0,4,0];  // 3D-print button pins
// ! select 3D-print array, use only one V comment out the others!

fn=24;  // holes quality, for 3D-print: 30..60

//============== Housing Dimension =================================
Hx=55; Hy=45; Hz=18; // xyz Housing dimension,
Ht=1.6;              // Ht:panel thickness
HDA=[Hx,Hy,Hz,Ht];   // Housing Dimension in Array

// =============== Module Parameters ==============================
//--x, y, z, hole d        // Beams 
BD=[3, 3, 3,  1.4];        // Beams panel dimension 
//  x, y, hight screw hole // SunkHoles
PD=[Hx,Hy,Ht,   0.4];      // SunkHoles: Panel Dimension
//   x,  y,    d1, d2
GSd=[3.1,3.1, 1.5,2.5];    // SunkHoles: Gap x,y diameter d1,d2
AS=[1.0, Ht, 1.4];         // Arduino Support height, Wall, hole

//================ Module position & rotate =========================
//     ----translate-----     --rotate--   
//     x       y        z     x  y    z
Ptr01=[0,      0,       0,    0, 0,   0]; // Panel housing down
Ptr02=[0,      0,   Hz-Ht,    0, 0,   0]; // sunk Holes
Ptr03=[0,      0,  Hz-6.2,    0, 0,   0]; // Beams 
Ptr04=[5,      3,   Hz-Ht,    0, 0,   0]; // OLED
Ptr05=[41,     Ht,  Hz-Ht,    0, 0,   0]; // Button
Ptr06=[Hx-2.5, 27,      0,    0, 0, 180]; // Arduino NanoEvery
Ptr07=[0,       0,  Hz-Ht,    0, 0,   0]; // free

//================ Module View, Position, Parameters in Arrays =========
//  View  Position Parameters $fn       name         includes
A01=[V[0], Ptr01,  HDA          ]; // House        0_House.scad
A02=[V[2], Ptr03,  HDA,  BD,  fn]; // Beams4H      0_House.scad
A03=[V[1], Ptr02,  PD,  GSd,  fn]; // SunkHolesZ   0_House.scad
A10=[V[3], Ptr04,  Ht,       ,fn]; // OLED13       0_LCD-OLED-LED.scad
A11=[V[4], Ptr05,  Ht,       ,fn]; // Buttons3     0_ButtonSwitch.scad
A12=[V[5], Ptr06,  AS,       ,fn]; // ArduinoNanoE 0_MicroControler.scad

//============== Show Modules ===================================
ShowModule(); 
module ShowModule() {  //housing  modules
  difference() {
    union() {  
//----------- View    Position Parameters     $fn ----
        House(A01[0], A01[1],  A01[2]);             
      Beams4H(A02[0], A02[1],  A02[2],A02[3], A02[4]);
       OLED13(A10[0], A10[1],  A10[2],        A10[3]);
     Buttons3(A11[0], A11[1],  A11[2],        A11[3]);
    ArduNanoE(A12[0], A12[1],  A12[2],        A12[3]);
	}
//------------clipping modules-------------------------
   SunkHolesZ(A03[0], A03[1],  A03[2],A03[3], A03[4]);
      OLED13C(A10[0], A10[1],  A10[2],        A10[3]);
    Buttons3C(A11[0], A11[1],  A11[2],        A11[3]);
   ArduNanoEC(A12[0], A12[1],  A12[2],        A12[3]);
} }
// that's all
```

### Final note

This construction should prevent spagetti code in openSCAD, which otherwise can arise very fast, which was often the case with me.  

I hope this concept is helpful for openSCAD users.
