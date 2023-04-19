# Abaqus-Python-Cylinder-Script
# A code created as part of a dissertation that can be adjusted to create different parameters for a Cylindrical Abaqus Model

from part import *
from material import *
from section import *
from assembly import *
from step import *
from interaction import *
from load import *
from mesh import *
from optimization import *
from job import *
from sketch import *
from visualization import *
from connectorBehavior import *
import os
import matplotlib.pyplot as plt
import xyPlot
import displayGroupOdbToolset as dgo
import numpy as np


#Change work directory
os.chdir(r"C:\Users\tahah\Desktop\Uni Final Year\Dissertation")
# os.chdir(r"C:\Users\pinhosl3\Desktop\Projects\Cole")


#VERY IMPORTANT
session.journalOptions.setValues(replayGeometry=COORDINATE, recoverGeometry=COORDINATE)

base_dir = os.getcwd()
folder_prefix = "/Cylinder "
folder_number = 1
folder_path = base_dir + folder_prefix + str(folder_number)

while os.path.exists(folder_path):
    folder_number += 1
    folder_path = base_dir + folder_prefix + str(folder_number)

os.mkdir(folder_path)
os.chdir(folder_path)

   
radius = 400.0 #mm  
model_height = 1200 #mm
load = 1 #N/mm2
max_cycles = 25    


def TopologyCylinder(mesh_size):

    # start a new model
    Mdb()
        
    #Cylinder shape

    mdb.models['Model-1'].ConstrainedSketch(name='__profile__', sheetSize=200.0)
    mdb.models['Model-1'].sketches['__profile__'].CircleByCenterPerimeter(center=(
        0.0, 0.0), point1=(radius, 0.0))
    mdb.models['Model-1'].Part(dimensionality=THREE_D, name='Part-1', type=
        DEFORMABLE_BODY)
    mdb.models['Model-1'].parts['Part-1'].BaseSolidExtrude(depth=model_height, sketch=
        mdb.models['Model-1'].sketches['__profile__'])
    del mdb.models['Model-1'].sketches['__profile__']


    #Material
    mdb.models['Model-1'].Material(name='Material-1')
    mdb.models['Model-1'].materials['Material-1'].Elastic(table=((210000.0, 0.3), 
        ))
    


    #Creates Sections

    mdb.models['Model-1'].HomogeneousSolidSection(material='Material-1', name=
        'Section-1', thickness=None)
    mdb.models['Model-1'].parts['Part-1'].Set(cells=
        mdb.models['Model-1'].parts['Part-1'].cells.getSequenceFromMask(('[#1 ]', 
        ), ), name='Set-1')
    mdb.models['Model-1'].parts['Part-1'].SectionAssignment(offset=0.0, 
        offsetField='', offsetType=MIDDLE_SURFACE, region=
        mdb.models['Model-1'].parts['Part-1'].sets['Set-1'], sectionName=
        'Section-1', thicknessAssignment=FROM_SECTION)
        
        #Assembly
        
    mdb.models['Model-1'].rootAssembly.DatumCsysByDefault(CARTESIAN)
    mdb.models['Model-1'].rootAssembly.Instance(dependent=ON, name='Part-1-1', 
        part=mdb.models['Model-1'].parts['Part-1'])
    mdb.models['Model-1'].StaticStep(name='Step-1', previous='Initial')
    mdb.models['Model-1'].rootAssembly.Surface(name='Surf-1', side1Faces=
        mdb.models['Model-1'].rootAssembly.instances['Part-1-1'].faces.getSequenceFromMask(
        ('[#2 ]', ), ))
        
     #This creates the step
    mdb.models['Model-1'].StaticStep(initialInc=0.05, maxInc=0.05, maxNumInc=20, name='Step-1', 
            nlgeom=ON, previous='Initial')
            
        
     #Load
        
    mdb.models['Model-1'].Pressure(amplitude=UNSET, createStepName='Step-1', 
        distributionType=UNIFORM, field='', magnitude=load, name='Load-1', region=
        mdb.models['Model-1'].rootAssembly.surfaces['Surf-1'])
        
        
    #Boundary Conditions 


    mdb.models['Model-1'].rootAssembly.Set(faces=
        mdb.models['Model-1'].rootAssembly.instances['Part-1-1'].faces.getSequenceFromMask(
        ('[#2 ]', ), ), name='Set-3')
    mdb.models['Model-1'].DisplacementBC(amplitude=UNSET, createStepName='Step-1', 
        distributionType=UNIFORM, fieldName='', fixed=OFF, localCsys=None, name=
        'BC-1', region=mdb.models['Model-1'].rootAssembly.sets['Set-3'], u1=0.0, 
        u2=0.0, u3=UNSET, ur1=UNSET, ur2=UNSET, ur3=UNSET)
    mdb.models['Model-1'].rootAssembly.Set(faces=
        mdb.models['Model-1'].rootAssembly.instances['Part-1-1'].faces.getSequenceFromMask(
        ('[#4 ]', ), ), name='Set-4')
    mdb.models['Model-1'].DisplacementBC(amplitude=UNSET, createStepName='Step-1', 
        distributionType=UNIFORM, fieldName='', fixed=OFF, localCsys=None, name=
        'BC-2', region=mdb.models['Model-1'].rootAssembly.sets['Set-4'], u1=0.0, 
        u2=0.0, u3=0.0, ur1=0.0, ur2=0.0, ur3=0.0)
        
        #Mesh

    mdb.models['Model-1'].parts['Part-1'].seedPart(deviationFactor=0.1, 
        minSizeFactor=0.1, size=mesh_size)
    mdb.models['Model-1'].parts['Part-1'].generateMesh()
    mdb.models['Model-1'].rootAssembly.regenerate()

    #Optimisation

    #cycles
    #final_volume
        
       
    mdb.models['Model-1'].TopologyTask(name='Task-1', region=MODEL)
    mdb.models['Model-1'].optimizationTasks['Task-1'].SingleTermDesignResponse(
        drivingRegion=None, identifier='VOLUME', name='D-Response-1', operation=SUM
        , region=MODEL, stepOptions=())
    mdb.models['Model-1'].optimizationTasks['Task-1'].setValues(
            freezeBoundaryConditionRegions=ON)    
    mdb.models['Model-1'].optimizationTasks['Task-1'].SingleTermDesignResponse(
        drivingRegion=None, identifier='STRAIN_ENERGY', name='Strain', operation=
        SUM, region=MODEL, stepOptions=())
    mdb.models['Model-1'].optimizationTasks['Task-1'].ObjectiveFunction(name=
        'Objective-1', objectives=((OFF, 'Strain', 1.0, 0.0, ''), ))
    mdb.models['Model-1'].optimizationTasks['Task-1'].OptimizationConstraint(
        designResponse='D-Response-1', name='volume constraint', restrictionMethod=
        RELATIVE_LESS_THAN_EQUAL, restrictionValue=0.3)
    mdb.OptimizationProcess(dataSaveFrequency=OPT_DATASAVE_SPECIFY_CYCLE, 
        description='', maxDesignCycle=max_cycles, model='Model-1', name='Opt-Process-1', 
        odbMergeFrequency=2, prototypeJob='Opt-Process-1-Job', saveInitial=False, 
        task='Task-1')
    mdb.optimizationProcesses['Opt-Process-1'].Job(atTime=None, 
        getMemoryFromAnalysis=True, memory=90, memoryUnits=PERCENTAGE, model=
        'Model-1', multiprocessingMode=DEFAULT, name='Opt-Process-1-Job', numCpus=1
        , numGPUs=0, queue=None, waitHours=0, waitMinutes=0)
    
    mdb.optimizationProcesses['Opt-Process-1'].setValues(dataSaveFrequency=
    OPT_DATASAVE_EVERY_CYCLE)
    mdb.optimizationProcesses['Opt-Process-1'].jobs['Opt-Process-1-Job'].setValues()
    
    #Job

    mdb.Job(atTime=None, contactPrint=OFF, description='', echoPrint=OFF, 
        explicitPrecision=SINGLE, getMemoryFromAnalysis=True, historyPrint=OFF, 
        memory=90, memoryUnits=PERCENTAGE, model='Model-1', modelPrint=OFF, 
        multiprocessingMode=DEFAULT, name='Job-1', nodalOutputPrecision=SINGLE, 
        numCpus=1, numGPUs=0, queue=None, resultsFormat=ODB, scratch='', type=
        ANALYSIS, userSubroutine='', waitHours=0, waitMinutes=0)

    mdb.optimizationProcesses['Opt-Process-1'].submit()
    a = mdb.models['Model-1'].rootAssembly
    session.viewports['Viewport: 1'].setValues(displayedObject=a)
    mdb.optimizationProcesses['Opt-Process-1'].waitForCompletion()
    print("Model with mesh_size = %s is completed" % str(mesh_size))


def PostProcessOptimization():
    mdb.CombineOptResults(optResultLocation=folder_path + "/" + str(mesh_size) + "/" + 'Opt-Process-1', 
        includeResultsFrom=LAST, optIter=ALL, models=ALL, steps=('Step-1', ), 
        analysisFieldVariables=('DRESP_STIFF', 'MAT_PROP_NORMALIZED', 'S', 'U', 'D_STIFF__DENSITY'))
    session.mdbData.summary() #useless
    o6 = session.openOdb(name=folder_path + "/" + str(mesh_size) + "/" + 'Opt-Process-1/TOSCA_POST/Opt-Process-1-Job_post.odb')
    
    session.viewports['Viewport: 1'].setValues(displayedObject=o6)
    
    session.viewports['Viewport: 1'].odbDisplay.setValues(viewCutNames=('Opt_Surface', ), viewCut=OFF)
    session.viewports['Viewport: 1'].odbDisplay.setValues(viewCut=ON)
    
    session.viewports['Viewport: 1'].odbDisplay.setPrimaryVariable(
        variableLabel='MAT_PROP_NORMALIZED', outputPosition=ELEMENT_CENTROID, )
    session.viewports['Viewport: 1'].odbDisplay.display.setValues(
        plotState=CONTOURS_ON_DEF)
    session.viewports['Viewport: 1'].odbDisplay.viewCuts['Opt_Surface'].setValues(
        showModelAboveCut=False)
        
    # session.viewports['Viewport: 1'].odbDisplay.setFrame(step=1, frame=4 )
    # session.viewports['Viewport: 1'].view.setValues(session.views['Front'])
    # session.viewports['Viewport: 1'].view.setValues(session.views['Back'])
    # session.viewports['Viewport: 1'].view.setValues(session.views['Top'])
    session.viewports['Viewport: 1'].view.setValues(session.views['Bottom'])
    # session.viewports['Viewport: 1'].view.setValues(session.views['Left'])
    # session.viewports['Viewport: 1'].view.setValues(session.views['Right'])
    
    for cut_percentage in np.arange(0.1,1.0,0.1):
        session.viewports['Viewport: 1'].view.fitView()
        session.viewports['Viewport: 1'].odbDisplay.viewCuts['Opt_Surface'].setValues(value=cut_percentage)
        session.printToFile(fileName='%s_bottom.jpeg' % str(round(cut_percentage,1)), format=PNG, canvasObjects=(session.viewports['Viewport: 1'], ))
    
    session.viewports['Viewport: 1'].view.setValues(session.views['Iso'])
    for cut_percentage in np.arange(0.1,1.0,0.1):
        session.viewports['Viewport: 1'].view.fitView()
        session.viewports['Viewport: 1'].odbDisplay.viewCuts['Opt_Surface'].setValues(value=cut_percentage)
        session.printToFile(fileName='%s_3D.jpeg' % str(round(cut_percentage,1)), format=PNG, canvasObjects=(session.viewports['Viewport: 1'], ))
    
    session.viewports['Viewport: 1'].view.setValues(nearPlane=2495.19,farPlane=4505.98, width=2710.43, height=1104, cameraPosition=(-1399.93, 
        -2118.53, 3009.58), cameraUpVector=(0.141946, 0.907734, 0.394805),cameraTarget=(-21.9662, -38.5189, 638.108))
    for cut_percentage in np.arange(0.1,1.0,0.1):
        session.viewports['Viewport: 1'].view.fitView()
        session.viewports['Viewport: 1'].odbDisplay.viewCuts['Opt_Surface'].setValues(value=cut_percentage)
        session.printToFile(fileName='%s_custom_view.jpeg' % str(round(cut_percentage,1)), format=PNG, canvasObjects=(session.viewports['Viewport: 1'], ))
        
        
for mesh_size in [25.0]:
    
    os.mkdir(folder_path + "/" + str(mesh_size))
    os.chdir(folder_path + "/" + str(mesh_size))
    TopologyCylinder(mesh_size)
    PostProcessOptimization()
