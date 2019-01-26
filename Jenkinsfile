/* 
* Copyright (c) 2019 and Confidential to Carefirst All rights reserved.  
*/ 

import groovy.json.*
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import hudson.Util;

pipeline {
    agent any

    options {
      timestamps()
      timeout(time: 15, unit: 'MINUTES')
      withCredentials([
        usernamePassword(credentialsId: 'PDM_JFrogRepository', 
            passwordVariable: 'P@ssw0rd_pega', 
            usernameVariable: 'pega_admin')
        /*usernamePassword(credentialsId: 'puneeth_export', 
            passwordVariable: 'rules', 
            usernameVariable: 'puneeth_export')*/
        ])
    }
    stages {
// in a declarative pipeline
        stage('Trigger Building') {
		                    steps {
						executeModuleScripts('build') // local method, see at the end of this script
				          }
				  }
}
}
// at the end of the file or in a shared library
void executeModuleScripts(String operation) {
 item='applicationName'
	    String deva='DevA'
	    String devb='DevB'
	    String sita='SitA'
	    String sitb='SitB'
	    String uat='UAT'
	    String tt='TT'
            String prod='PROD'
	    String AppName='applicationName'
	    String AppVersion='applicationVersion'
	    def EnvList = []
	    String a = "${env.JOB_NAME}".split('/')[0] as String
	    String AppPath= "/home/pegacoeadm/" + "${a}"+".json"
	    println "$AppPath"
	    //def inputFile = new File("/home/pegacoeadm/test1.json")
	    def inputFile = new File("${AppPath}")
	    def InputJSON = new JsonSlurperClassic().parseFile(inputFile, 'UTF-8')
            def appname,appversion,stgs,devbstgs,devastgs,sitastgs,sitbstgs,uatstgs,ttstgs,prodstgs
	    InputJSON.AppDetails.each { 
	    stgs=it."$item";
	    devbstgs=it."$devb";
	    devastgs=it."$deva";
	    sitastgs=it."$sita";
	    sitbstgs=it."$sitb";
	    uatstgs=it."$uat";
            ttstgs=it."$tt";
	    prodstgs=it."$prod";
            appname=it."$AppName";
	    appversion=it."$AppVersion";
	    println "${devastgs}";
	    if (devastgs) { 
	    
	    	//EnvList.add("${devastgs}")
		devastgs="http://svl-pgwasdb-d1:9101/pdmodevb"
		EnvList.add("DevA")
	    	}

	   if (devbstgs) {
	                   //EnvList.add("${devbstgs}")
			   EnvList.add("DevB")

			      }
	   if (sitastgs) {
	                  //EnvList.add("${sitastgs}")
                         EnvList.add("SITA")
			      }

	   if (sitbstgs) {
	                  //EnvList.add("${sitbstgs}")
			  EnvList.add("SITB")

			             }

	   if (uatstgs) {
	                 //EnvList.add("${uatstgs}")
			 EnvList.add("UAT")

			            }
	   if (ttstgs) {
	                EnvList.add("TT")

			           }

	   if (prodstgs) {
	                //EnvList.add("${prodstgs}")
			EnvList.add("PROD")

			           }



		}
	    //InputJSON.each{  k, v ->println v }
            def allModules = EnvList.collect()
	    allModules.each { module ->  String action = "${operation}:${module}"  
           echo("---- ${action.toUpperCase()} ----") 
	   String command = "npm run ${action} -ddd"                   
           // here is the trick           
           script {

																								                stage(module) {
			if (module == 'DevA') {
			String PEGA_DEV_1 = "${devastgs}".split('/')[0] as String
			String PEGA_DEV_2 = "${devastgs}".split('/')[2] as String
			String PEGA_DEV = "${PEGA_DEV_1}"+"//" + "${PEGA_DEV_2}"+"/"+"pdmodevb/PRRestService/PegaUnit/Rule-Test-Unit-Case/pzExecuteTests"
			println "${PEGA_DEV}"
			                           echo 'Initiating UT...'
						   withEnv(['TESTRESULTSFILE="TestResult.xml"']) {
						   sh "./gradlew executePegaUnitTests -PtargetURL=${PEGA_DEV} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE}"
						   String buildStatus = currentBuild.currentResult
						   println "${buildStatus}"
						   //sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PbuildStatus=${buildStatus}"
						   junit '**/TestResult.xml'
						   script {
							def userInput
						    if (currentBuild.result != null) {
                                                    try{
						    sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PbuildStatus=waiting"
						     userInput = input(message: 'Unit Tests have failed, would you like to abort the pipeline?')
						     println "${userInput}"
						     currentBuild.result = "SUCCESS"
						     buildStatus = "SUCCESS,Approved"
						     sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PbuildStatus=${buildStatus}"
						     println "Took ${currentBuild.startTimeInMillis}"
						     }catch(err) { // input false
						         echo "This Job has been Aborted"
                                                         currentBuild.result = 'UNSTABLE'
							 buildStatus = 'Aborted'
							 sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PbuildStatus=${buildStatus}"
							 }
						     }
						     }
						     }
						     }
				}
                  }
	        }
				
}	

