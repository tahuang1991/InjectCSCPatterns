# Pattern injection test with emulator board
Document and code to generate comaprator digi and CLCT+LCT information for patten injection test


## Generate txt file with CSC L1 trigger emulator

  - step1: checkout cmssw and CSC L1 trigger emulator. 
  ```
cmsrel CMSSW_12_5_0_pre4
cd CMSSW_12_5_0_pre4/src
cmsenv
git cms-addpkg L1Trigger/CSCTriggerPrimitives
  ```
  Here is the link to CSCTrigger emulator in CMSSW:  https://github.com/cms-sw/cmssw/tree/master/L1Trigger/CSCTriggerPrimitives
  
  And here is the part for CLCT emulation: https://github.com/cms-sw/cmssw/blob/master/L1Trigger/CSCTriggerPrimitives/src/CSCCathodeLCTProcessor.cc 
  - step2: apply the changes to CSC L1 trigger emulator and compile cmssw. 
  ```
git clone https://github.com/tahuang1991/InjectCSCPatterns.git
cp InjectCSCPatterns/CSCTriggerPrimitives/src/* L1Trigger/CSCTriggerPrimitives/src/
cp InjectCSCPatterns/CSCTriggerPrimitives/interface/*  L1Trigger/CSCTriggerPrimitives/interface
scram b -j 9
  ```
  One thing worth attention is that the above commands are to over-write the old files in L1Trigger/CSCTriggerPrimitives.  If L1Trigger/CSCTriggerPrimitives package is updated with some changes that InjectCSCPatterns/CSCTriggerPrimitives does not include, then over-write the old files may not work.  
  
  - step3: run CSC L1 trigger emulation to get txt file. Replace the inputFiles with sample you want to process and set maxEvents to the number of events you need
  ```
  cd L1Trigger/CSCTriggerPrimitives/test
  rm ComparatorDigi_CLCT_ME*.txt
  cmsRun runCSCTriggerPrimitiveProducer_cfg.py mc=True run3=True inputFiles="file:/afs/cern.ch/user/t/tahuang/public/RelValSample1000GeVTest/27a95851-6358-485b-b15b-619f3404d795.root" maxEvents=10 saveEdmOutput=False l1=True runME11ILT=True runCCLUTOTMB=True
  ```

  
The output files generated from CSC L1 trigger emulator include:
  - ComparatorDigi_CLCT_ME11.txt for ME11 chamber type, with CCLUT and GEMCSC algorithm on
  - ComparatorDigi_CLCT_ME21.txt for ME21 chamber type, with CCLUT on
  - ComparatorDigi_CLCT_ME3141.txt for ME3141 chamber type, with CCLUT on

Three example txt files from 10 events are included under data/

Everytime you run above program,  it would append the new printouts to the exist output files. Make sure that old files are removed if you want to creat new txt files

#### Files changed to print out comparator digis, GEM clusters, CLCT and LCTs
The printout code changes is summarized in the following commits:
  -  [a881941d8b459926564a0873621c55aae9090ca0](https://github.com/tahuang1991/InjectCSCPatterns/commit/a881941d8b459926564a0873621c55aae9090ca0)
  -  [c3fd3ace655739a7d86e099e63b35dc412310338](https://github.com/tahuang1991/InjectCSCPatterns/commit/c3fd3ace655739a7d86e099e63b35dc412310338)
 
The safe way to include printout code is applying the printout code changes to L1Trigger/CSCTriggerPrimitives by hand.The reason is because with newer CMSSW version, the CSC trigger emulator code might be modified for other reasons and you do not want to overwrite these changes.

What you need to do is apply the changes in CSCTriggerPrimitives/interface/CSCCathodeLCTProcessor.h, CSCTriggerPrimitives/src/CSCCathodeLCTProcessor.cc, CSCTriggerPrimitives/src/CSCMotherboard.cc, CSCTriggerPrimitives/src/CSCGEMMotherboard.cc to the corresponding files under L1Trigger/CSCTriggerPrimitives/

## Txt file from CSC L1 trigger emulator conventions
The typical printout for one chamber with comparator digi from one event is showed in the following:

Start with "CSCChamber with Comparatordigi:" + detector information (endcap=1 means postive endcap and =2 means negative endcap)
>```
>CSCChamber with Comparatordigi: (end,station,ring,chamber) = 1, 2, 1, 7  
>```

Comparator digi part: ranked by BX and layer
>```
>Comparatordigi BX 7 Layer 1 halfstrip 67 
>Comparatordigi BX 7 Layer 2 halfstrip 67
>Comparatordigi BX 7 Layer 4 halfstrip 67
>Comparatordigi BX 7 Layer 5 halfstrip 67
>Comparatordigi BX 8 Layer 0 halfstrip 67
>Comparatordigi BX 8 Layer 3 halfstrip 67
>```

CLCT part: CLCTs in this chamber, up to two CLCTs per BX, ranked by BX
>```
>CSC CLCT #1: Valid = 1 BX = 7 Run-2 Pattern = 10 Run-3 Pattern = 4 Quality = 6 Comp Code 4095 Bend = 1  
>Slope = 0 CFEB = 2 Strip = 2 KeyHalfStrip = 66 KeyQuartStrip = 132 KeyEighthStrip = 265
>```

GEM clusters from GE11 in same endcap and with same chamber number as CSC: ranked by layer and BX
>```
>GEMCluster in GE11 layer1: bx 8 gemPad 26 size 3 roll 6 converted into CSC coordination: wiregroup 15 halfstrip 21
>GEMCluster in GE11 layer1: bx 8 gemPad 87 size 2 roll 6 converted into CSC coordination: wiregroup 15 halfstrip 58
>GEMCluster in GE11 layer2: bx 8 gemPad 43 size 3 roll 6 converted into CSC coordination: wiregroup 15 halfstrip 31
>```

LCT part: LCTs in this chamber, up to two LCTs per BX,  ranked by BX
>```
>CSC LCT #1: Valid = 1 BX = 8 Run-2 Pattern = 10 Run-3 Pattern = 4 Quality = 3 Bend = 1 Slope = 0   
>KeyHalfStrip = 66 KeyQuartStrip = 132 KeyEighthStrip = 265 KeyWireGroup = 104 Type (SIM) = 1 MPC Link = 0
>```


## Generate txt file with GEMCode

To be included as an optional way to dump comparator digis and CLCT+LCTs to txt file  from GEMCode package. 

