This document details how to install and use GEANT4 on Ubuntu 22.04. 

Installation and testing

1. Download the GEANT4 source code from the GEANT4 website, ensuring you download the code for the right OS. 
2. This will download in a .zip file, so will need to be unpacked. It would be useful to extract this into a seperate directory, called something like 'software'.
3. Make another subfolder in software called geant4. 
4. Extract the contents of the download into this folder. 
5. Once extracted move into the folder that's created, this will be called something like 'geant4.x.x.x.x'.
6. Note, this is a CMAKE project. 
7. Before compiling this, you will need to make sure you've installed the required packages in Ubuntu, this can be done with the following command 
'sudo apt install cmake cmake-curses-gui gcc g++ libexpat1-dev qt5-default libxmu-dev libmotif-dev'. Note the qt5 package will likely throw and error 
as this is no longer included on modern Ubuntu distributions. The workaround can be achieved by installing three other packages, easily found by googling the error code. 
8. Then make an additional folder within the geant4.x.x.x.x folder called build.
9. Move into the build folder. 
10. Then use ccmake .. to open the cmake gui to prep the config. 
11. Press c to start the configure. 
12. Ignore the initial warning, provided there's no errors. 
13. Change the installation prefix to '/home/NAME/software/geant4/geant4.x.x.x.x-install', this will create a new folder called install to store the installation.
14. Here you can choose to install either multithread or single thread, for simplicity it would be worth choosing single thread to start. 
15. Specify ON for GEANT4_INSTALL_DATA, GEANT4_USE_QT, GEANT4_RAYTRACER_X11 and GEANT4_USE_SYSTEM_EXPAT.
16. Press c for configure. You may get an error saying some of the QT packages are not found, you can install these seperately but the code should compile either way. 
17. Press c for configure again. 
18. Press g for generate. 
19. Once complete check the contents of the build folder to make sure the code has compiled. 
20. Hit make to begin the build. 
21. This process will likely take a long time to complete so allow the installation to run through. 
22. It may also download some additinal pacakages so make sure you have access to the internet at this point. 
23. Use the command make install to install the programme.
24. Check the contents of the install folder that you made previously, there should be folders names bin, include, lib and share. If these are there the installation completed correctly. 
25. Move into the share folder, then the Geant4.x.x.x.x folder, then geant4make. 
26. There should be a file in here called geant4make.sh, at this point you want to source this file using the command '. geant4make.sh'. This will set the tasks and the libraries from this file. 
27. Move back into the geant4.x.x.x.x folder and move back into the examples folder. There's some simple examples in here. 
28. Move into the basic folder, then move into the B1 folder - this contains the easiest examples to run. 
29. This is also a cmake project so make another folder in here called build and move into this new folder. 
30. Type cmake .. to make sure the code compiles. 
31. Type make.
32. Run this basic example using the command ./exampleB1.
33. At this point you can hit the play button at the top of the output to generate particles. 
34. Neutral particles are shown in green, negative in red and positive particles in blue. 
35. If you use the command '/run/beamOn 100' in the command prompt at the bottom of the output you will generate 100 particles. 

Creating your first project, including sim.cc

1. Th first step in creating a new project is to reference the build files from the previous installation, this can be completed using the command
'. software/geant4/geant.x.x.x.x-install/share/Geant4/geant4make/geant4make.sh'
2. Then, from the top level directory, create a new folder called sim, this will be used to store all of the source code for the new project.
3. Moving into the new folder, start by creating a file called CMakeLists.txt (just a normal text file). The contents of this file should read: 
	
	cmake_minimum_required(VERSION 2.6 FATAL_ERROR) //this is used to make sure the required cmake package is installed. 

	project(Simulation) //this is used to name the project, here we've called it simulation.

	find_package(Geant4 REQUIRED ui_all vis_all) //this is used to tell cmake to find GEANT4 (noting it's required), ui_all and vis_all to enable visualisation and ui interaction. 

	include(${Geant4_USE_FILE}) //this includes the contents that was previously sourced using the command in 1.

	file(GLOB sources ${PROJECT_SOURCE_DIR}/*.cc) //this is used to take all the source files (.cc) from the PROJECT_SOURCE_DIR and compile them using gcc.
	file(GLOB headers ${PROJECT_SOURCE_DIR}/*.hh) //this is used to take all the header files (.hh) from the PROJECT_SOURCE_DIR and compile them using gcc. 

	add_executable(sim sim.cc ${sources} ${headers}) //this adds the executable called sim, inserts the main file with the main source code in - called sim.cc, 
then includes the sources and variables headers

	target_link_libraries(sim ${Geant4_LIBRARIES}) //this links the libraries required in geant4 and links them to the executable sim. 

	add_custom_target(Simulation DEPENDS sim) //this links the simulation to sim.

4. Next create a source file called sim.cc in the same directory. 
5. The content of the source file should read: 

	#include <iostream> //the contents of these includes ensures the references to the external files are in place.

	#include "G4RunManager.hh"
	#include "G4UImanager.hh"
	#include "G4VisManager.hh"
	#include "G4VisExecutive.hh"
	#include "G4UIExecutive.hh"

	#include "construction.hh" //this is how we call the contents of construction.hh and construction.cc.
	#include "physics.hh" //here we're including the contents of both the physics header and physics contructor files.
	#include "action.hh" //include the contents of the action header file.

	int main(int argc, char** argv) //this inititates the main routine for the programme.
	{
		G4RunManager* runManager = new G4RunManager; //this inititates the heart of Geant4 and calls it runManager.

		runManager->SetUserInitialization(new MyDetectorConstruction()); //this is where we tell sim.cc to use the volume declared in construction.cc.
		runManager->SetUserInitialization(new MyPhysicsList()); //this is where we initialise the physics list in the main constructor.
		runManager->SetUserInitialization(new MyActionInitialization()); //include the contents of the action constructor.

		runManager->Initialize(); //this command is used to include the additional functionality within the simulation from other files. 

		G4UIExecutive *ui = new G4UIExecutive(argc, argv); //this bit allows for the user to interact with the simulation, the arguments come from the programme input parameters. 

		G4VisManager *visManager = new G4VisExecutive(); //this bit allows for the visualisation within the simulation.
		visManager->Initialize(); //this initialises the visManager.

		G4UImanager *UImanager = G4UImanager::GetUIpointer(); //this is in place to initialise the UI manager.

		UImanager->ApplyCommand("/vis/open OGL"); //these commands are used to initialise the simulation, set the start point for the particle gun, draw volumes, etc. 
		UImanager->ApplyCommand("/vis/viewer/set/viewpointVector 1 1 1"); //this is used to change the initial position that the simulation opens from. 
		UImanager->ApplyCommand("/vis/drawVolume"); //here we're telling Geant4 to draw the full volume that's previously defined. 
		UImanager->ApplyCommand("/vis/viewer/set/autoRefresh true"); //here you ask Geant4 to update each time you create an event
		UImanager->ApplyCommand("/vis/scene/add/trajectories smooth"); //here you ask Geant4 to include the drawing of the particle path

		ui->SessionStart(); //this starts the session from the G4UIExecutive command.

		return 0; //this closes off the programme.
	}

6. Make a build folder under the sim folder. 
7. From inside this folder run the command cmake .. to compile the outlined source code. 
8. Run the command make and debug any errors, this will likely throw issues with which files have been included.
9. Run the simulation with the ./sim command and the Geant4 output should be displayed.

Adding detector construction, including construction.cc and construction.hh

1. Create two new files in the sim folder called construction.cc and construction.hh.
2. The contents of these files should read, respectively:

#include "construction.hh" //this is calling to the appropriate header file. 

MyDetectorConstruction::MyDetectorConstruction() //defining the constructor.
{}

MyDetectorConstruction::~MyDetectorConstruction() //defining the destructor. 
{}

G4VPhysicalVolume *MyDetectorConstruction::Construct() //this is defining the function from the header file. 
{
	G4NistManager *nist = G4NistManager::Instance(); //this is pulling from the existing materials that are defined a in GEANT4. 
	
	G4Material *SiO2 = new G4Material("SiO2", 2.201*g/cm3, 2); 
	SiO2->AddElement(nist->FindOrBuildElement("Si"), 1);
	SiO2->AddElement(nist->FindOrBuildElement("O"), 2);
	
	G4Material *H2O = new G4Material("H2O", 1.000*g/cm3, 2);
	H2O->AddElement(nist->FindOrBuildElement("H"), 2);
	H2O->AddElement(nist->FindOrBuildElement("O"), 1);
	
	G4Element *C = nist->FindOrBuildElement("C");
	
	G4Material *Aerogel = new G4Material("Aerogel", 0.200*g/cm3, 3);
	Aerogel->AddMaterial(SiO2, 62.5*perCent);
	Aerogel->AddMaterial(H2O, 37.4*perCent);
	Aerogel->AddElement(C, 0.1*perCent);
	
	G4double energy[2] = {1.239841939*eV/0.9, 1.239841939*eV/0.2};
	G4double rindexAerogel[2] = {1.1, 1.1};
	G4double rindexWorld[2] = {1.0, 1.0};
	
	G4MaterialPropertiesTable *mptAerogel = new G4MaterialPropertiesTable();
	mptAerogel->AddProperty("RINDEX", energy, rindexAerogel, 2);
	
	Aerogel->SetMaterialPropertiesTable(mptAerogel);
	
	G4Material *worldMat = nist->FindOrBuildMaterial("G4_AIR"); //this is the bit where you're looking for air to fill the world material. 
	
	G4MaterialPropertiesTable *mptWorld = new G4MaterialPropertiesTable();
	mptWorld->AddProperty("RINDEX", energy, rindexWorld, 2);
	
	worldMat->SetMaterialPropertiesTable(mptWorld);

	G4Box *solidWorld = new G4Box("solidWorld", 0.5*m, 0.5*m, 0.5*m); //here we are defining the physical space in which the detector will sit and calling it solidWorld. Note the standard unit in Geant4 is mm, so we need to define lengths in m.

	G4LogicalVolume *logicWorld = new G4LogicalVolume(solidWorld, worldMat, "logicWorld"); //we are putting the air material into our box here and are calling it logicWorld.

	G4VPhysicalVolume *physWorld = new G4PVPlacement(0, G4ThreeVector(0., 0., 0.), logicWorld, "physWorld", 0, false, 0, true); //here we define our physical volume. The variables passed are there to ensure rotation, centre, the logic volume, name, mother volume, boolean operation, copy number (number of logical volume inserts), and whether Geant$ should check for overlaps.
	
	G4Box *solidRadiator = new G4Box("solidRadiator", 0.4*m, 0.4*m, 0.01*m);
	
	G4LogicalVolume *logicRadiator = new G4LogicalVolume(solidRadiator, Aerogel, "logicalRadiator");
	
	G4VPhysicalVolume *physRadiator = new G4PVPlacement(0, G4ThreeVector(0., 0., 0.25*m), logicRadiator, "physRadiator", logicWorld, false, 0, true);
	
	G4Box *solidDetector = new G4Box("solidDetector", 0.005*m, 0.005*m, 0.01*m);
	
	logicDetector = new G4LogicalVolume(solidDetector, worldMat, "logicalDetector");
	
	for(G4int i = 0; i < 100; i++){
	
		for(G4int j = 0; j < 100; j++){
			G4VPhysicalVolume *physDetector = new G4PVPlacement(0, G4ThreeVector(-0.5*m+(i+0.5)*m/100, -0.5*m+(j+0.5)*m/100, 0.49*m), logicDetector, "physDetector", logicWorld, false, j+i*100, true);
		}
	}

	return physWorld; //here we are returning our top level mother volume. 
}

void MyDetectorConstruction::ConstructSDandField()
{
	MySensitiveDetector *sensDet = new MySensitiveDetector("SensitiveDetector");
	
	logicDetector->SetSensitiveDetector(sensDet);
}

and 

#ifndef CONSTRUCTION_HH //this bit is used to make sure it's not included several times. 
#define CONSTRUCTION_HH //this bit is used to set out the definition.

#include "G4VUserDetectorConstruction.hh" //these are here to ensure we include the detail from the classes that have been called in the .cc file. 
#include "G4VPhysicalVolume.hh"
#include "G4LogicalVolume.hh"
#include "G4Box.hh"
#include "G4PVPlacement.hh"
#include "G4NistManager.hh"
#include "G4SystemOfUnits.hh" //this is the bit where we can use different units that are predefined in Geant4, rather then just the default units. 

#include "detector.hh"

class MyDetectorConstruction : public G4VUserDetectorConstruction //this is where you're defining the class, here the name is MyDetectorConstruction. It is inherited from the G4UserDetectionConstruction class. 
{
public:
	MyDetectorConstruction(); //this is the constructor.
	~MyDetectorConstruction(); //this is the destructor. 

	virtual G4VPhysicalVolume *Construct(); //this is the return value of this function, the call to Construct is the piece that creates the physical construction. It is a virtual construction. 
	
private:
	G4LogicalVolume *logicDetector;
	virtual void ConstructSDandField();
};

#endif //this bit ends the header file. 

3. Each time you add a new file to the simulation you need to run cmake .. again to include these additions. 
4. From there, make the simulation using the make command. 
5. The execute the simulation using the ./sim command. 

Implementing Physics List

1. Start by creating two new files called phisics.cc and physics.hh.
2. The code for each of these should read, respectively: 

#include "physics.hh" //insert our physics file

MyPhysicsList::MyPhysicsList() //include the constructor
{
	RegisterPhysics (new G4EmStandardPhysics()); //define which physics you want to include, here standard EM (not hadronic) physics
	RegisterPhysics (new G4OpticalPhysics()); //then here include optical physics
}

MyPhysicsList::~MyPhysicsList() //this is the piece for the destructor, for now nothing included here
{}

#ifndef PHYSICS_HH //this bit is defining the list if it's not already defined.
#define PHYSICS_HH

#include "G4VModularPhysicsList.hh" //we're calling details from this public library so this needs to be included here
#include "G4EmStandardPhysics.hh"
#include "G4OpticalPhysics.hh"

class MyPhysicsList : public G4VModularPhysicsList //this uses the class name and inherits publically from G4VModularPhysics list. 
{
public: //you have to define your files as public so the other files can access them.
	MyPhysicsList(); //constructor
	~MyPhysicsList(); //destructor
	//we could write the definition for the physics list here but for consistency, we put the definition in physics.cc (the construction file)
};

#endif //this ends the initial if statement.

Generating Particles

1. Begin this process by creating two further files called action.cc and action.hh.
2. These should read respectively:

#include "action.hh" //include the action constructor file. 

MyActionInitialization::MyActionInitialization() //constructor
{}

MyActionInitialization::~MyActionInitialization() //destructor
{}

void MyActionInitialization::Build() const //outlining the function that's declared in the header file
{
	MyPrimaryGenerator *generator = new MyPrimaryGenerator(); //define the initital generator
	SetUserAction(generator); //command required to set the generator
	
	MyRunAction *runAction = new MyRunAction();
	SetUserAction(runAction);
}

#ifndef ACTION_HH //as usual here we're declaring these if they don't already exist
#define ACTION_HH

#include "G4VUserActionInitialization.hh" //inherit properties from the previously defined class

#include "generator.hh" //include the contents from the generator file
#include "run.hh"

class MyActionInitialization : public G4VUserActionInitialization //class definition and where it's inherited from.
{
public:
	MyActionInitialization(); //constructor
	~MyActionInitialization(); //destructor

	virtual void Build() const; //defining the virtual function from the class G4VUserActionInitialization
};

#endif //end the if statement

3. From there, create a further two files called gnerator.cc and generator.hh, these should read:

#include "generator.hh" //include header file

MyPrimaryGenerator::MyPrimaryGenerator() //constructor
{
	fParticleGun = new G4ParticleGun(1); //define the particle gun with the number of particles per event
}

MyPrimaryGenerator::~MyPrimaryGenerator() //destructor
{
	delete fParticleGun; //delete the particle gun
}

void MyPrimaryGenerator::GeneratePrimaries (G4Event *anEvent) //use the function defined elsewhere, G4Event is used to obtain input parameter
{
	G4ParticleTable *particleTable = G4ParticleTable::GetParticleTable(); //here we're defining what sort of particles we can to generate, here pulling from ParticleTable
	G4String particleName = "proton"; //defining our particle
	G4ParticleDefinition *particle = particleTable->FindParticle("proton"); //looking it up in the particle table

	G4ThreeVector pos(0.,0.,0.); //defining the particle start position and start momentum direction
	G4ThreeVector mom(0.,0.,1.);

	fParticleGun->SetParticlePosition(pos); //feed the starting position and momentum positions and directions to the particle gun
	fParticleGun->SetParticleMomentumDirection(mom);
	fParticleGun->SetParticleMomentum(100.*GeV); //set the size of the particle momentum
	fParticleGun->SetParticleDefinition(particle); //set the type of particle in question

	fParticleGun->GeneratePrimaryVertex(anEvent); //tell Geant4 to generate the primary vertex and hand over the variable anEvent
}

#ifndef GENERATOR_HH //same as before
#define GENERATOR_HH

#include "G4VUserPrimaryGeneratorAction.hh" //inherit the class from the Geant4 libraries

#include "G4ParticleGun.hh" //including the other pieces that will be used here
#include "G4SystemOfUnits.hh"
#include "G4ParticleTable.hh"


class MyPrimaryGenerator : public G4VUserPrimaryGeneratorAction //naming the class and saying where you're getting the contents from
{
public:
        MyPrimaryGenerator();
        ~MyPrimaryGenerator();

        virtual void GeneratePrimaries(G4Event*); //create the primaries that will be used by the action initialization

private:
	G4ParticleGun *fParticleGun; //defining and naming the particle gun
};

#endif

