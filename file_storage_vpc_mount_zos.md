---

copyright:
  years: 2022
lastupdated: "2022-09-23"

keywords: zos, file share

subcollection: vpc

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:important: .important}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:note: .note}
{:preview: .preview}
{:external: target="_blank" .external}



# Mounting file shares on z/OS
{: #file-storage-vpc-mount-zos}

Use these instructions to connect a z/OS-based {{site.data.keyword.cloud}} Compute Instance to a Network File System (NFS) file share. 

## Before you begin
{: #fs-zos-prereq}

Before you mount {{site.data.keyword.filestorage_vpc_short}} file shares, you must create a [virtual server instance](/docs/vpc?topic=vpc-about-advanced-virtual-servers) in the same zone as the file share. After you create an instance, get the mount path of the file share from the mount target created. You need a mount path for mounting file shares.

Mount path information can be obtained from the **File share details** page in the UI, or through an API or CLI call to get the mount target information.
{: tip}

## Mount the file share
{: #fs-zos-mount-file-share}

Follow these steps to mount a file share on a z/OS host.

1. Create a `/u/ibmuser` directory.
   
   ```
   mkdir /u/ibmuser/test
   ```
   {: codeblock}

2. Start the NFS client. You can issue commands `D OMVS,P` to verify whether the NFS client is started or not. 

   * If the entry for NFS file system is in the `sys1.parmlib.bpxprm00`, you can safely skip this step. 

   * If the entry for NFS file system is not in the `sys1.parmlib.bpxprm00`, you need to complete the following steps to start the NFS client:

     a. To set up the NFS client, run the following job on your z/OS virtual server instance.
      
     ```
     //NFSCLNT  JOB CLASS=A,MSGCLASS=H,MSGLEVEL=(1,1),NOTIFY=&SYSUID.,      
     //         REGION=0M                                                   
     //*********************************************************************
     //* Configuration for the NFS client.                                 *
     //*********************************************************************
     //ALIAS    EXEC PGM=IDCAMS                                             
     //SYSPRINT DD SYSOUT=*                                                 
     //SYSIN    DD *                                                        
       DEF ALIAS(NAME(MVSCLNT) RELATE(CATALOG.VS01.SYS)) -                  
           CAT(CATALOG.VS01.MASTER)                                         
     /*                                                                     
     //DEFLOG1  EXEC PGM=IEFBR14                                  
     //SYSPRINT DD SYSOUT=*                                       
     //LOG1     DD DSN=MVSCLNT.LOG1,                              
     //         DISP=(NEW,CATLG),UNIT=3390,SPACE=(CYL,(2,5),RLSE),
     //         RECFM=VB,LRECL=137,BLKSIZE=6144,VOL=SER=OPEVS1    
     //*                                                          
     //DEFLOG2  EXEC PGM=IEFBR14                                  
     //SYSPRINT DD SYSOUT=*                                       
     //LOG2     DD DSN=MVSCLNT.LOG2,                              
     //         DISP=(NEW,CATLG),UNIT=3390,SPACE=(CYL,(2,5),RLSE),
     //         RECFM=VB,LRECL=137,BLKSIZE=6144,VOL=SER=OPEVS1    
     //*                                                          
     //DEFRACF  EXEC PGM=IKJEFT01                                 
     //SYSPRINT DD SYSOUT=*                            
     //SYSTSPRT DD SYSOUT=*                            
     //SYSTSIN  DD *                                   
       RDEFINE STARTED MVSNFSC.* UACC(NONE) +          
         STDATA(USER(SYSTEM) GROUP(SYS1) TRUSTED(YES))
                                                       
       SETR REFRESH RACLIST(STARTED)                   
     /*                                                
     //COPYSAMP EXEC PGM=IEBCOPY                       
     //SYSPRINT DD SYSOUT=*                            
     //SYSUT1   DD UNIT=SYSDA,SPACE=(TRK,(5,1))        
     //SYSUT2   DD UNIT=SYSDA,SPACE=(TRK,(5,1))        
     //SYSUT3   DD UNIT=SYSDA,SPACE=(TRK,(5,1))        
     //SYSUT4   DD UNIT=SYSDA,SPACE=(TRK,(5,1))        
     //INDD     DD DSN=SYS1.NFSSAMP,DISP=SHR           
     //OUTDD    DD DSN=SYS1.PROCLIB,DISP=SHR           
     //SYSIN    DD *                                   
       COPY I=INDD,O=OUTDD                             
       SELECT MEMBER=((GFSCPROC,MVSNFSC))              
     /*                                                
     ```
     {: codeblock}

     b. Add the following command to `SYS1.PARMLIB(BPXPRM00)` after `AUTOMNT`.
     
     ```
     FILESYSTYPE TYPE(NFS) ENTRYPOINT(GFSCINIT) ASNAME(MVSNFSC)
     
     ```
     {: codeblock}
 
     c. To start the NFS client address space `MVSNFSC`, issue the following commands from the console `/SDSF Log`.
    
     ```
     set OMVS=00
     ```
     {: codeblock}

     d. To verify that the NFS client is started, issue the commands `D OMVS,P`.
    
     If the principal `MVSNFSC` is in the list, the NFS client is started successfully.
     {: note}

3. Run the following commands from TSO to mount the file share on z/OS in ASCII format:

   ```
   mount type(NFS) filesystem(nfs02) mountpoint('/u/ibmuser/test') parm('<mount path>,xlat(y),vers(4)')
   ```
   {: codeblock}

   You can also run the following commands to mount the file share on z/OS in binary format:

   ```
   mount type(NFS) filesystem(nfs02) mountpoint('/u/ibmuser/test') parm('<mount path>,vers(4)')
   ```
   {: codeblock}

   For more information, see [Additional mount command examples](https://www.ibm.com/docs/en/zos/2.4.0?topic=examples-additional-mount-command){: external}.

4. To verify that mounting file shares to z/OS is successfully completed, you can run command `df` to display the directory `/u/ibmuser/test`.