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
	    String Approval_Status='Pending-Approval'
	    String Dev_Completed='Dev-Completed'
	    String productName='DevOps_Export'
	    String productVersion='01-01-01'
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
		//devastgs="http://svl-pgwasdb-d1:9101/pdmodevb"
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

			//Start of first Stage of pipeline.
			String PEGA_DEV_1 = "${devastgs}".split('/')[0] as String
			String PEGA_DEV_2 = "${devastgs}".split('/')[2] as String
			String PEGA_DEV = "${PEGA_DEV_1}"+"//" + "${PEGA_DEV_2}"+"/"+"pdmodevb/PRRestService/PegaUnit/Rule-Test-Unit-Case/pzExecuteTests"
			println "${PEGA_DEV}"
			 sh "./gradlew sendUpdateToPega -PbuildStatus='Dev-Started' -PDateFlag=Start -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PStageName='DevA'"
			                           echo 'Initiating UT...'
						   withEnv(['TESTRESULTSFILE="TestResult.xml"']) {
						   sh "./gradlew executePegaUnitTests -PtargetURL=${PEGA_DEV} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PproductName=${productName} -PproductVersion=${productVersion} --debug"
						   String buildStatus = currentBuild.currentResult
						   println "${buildStatus}"
						   junit '**/TestResult.xml'
						   script {
							def userInput
							echo "uuernt duration puneeth: ${currentBuild.durationString}"
						    if (currentBuild.result != null) {
                                                    try{


						    sh "ssh pegacoeadm@svl-pgwasda-d1 mkdir -p /var/tmp/CICD/${appname}"
						    sh "exit"
						    sh "scp ${WORKSPACE}/${TESTRESULTSFILE} pegacoeadm@svl-pgwasda-d1:/var/tmp/CICD/${appname}/${TESTRESULTSFILE}"
																				
						    sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PpegaUsername=puneeth_dops -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE} -PbuildStatus='Pending-Approval' -PStageName='DevA'"
						     userInput = input(message: 'Unit Tests have failed, would you like to abort the pipeline?')
						     println "${userInput}"
						     currentBuild.result = "SUCCESS"
						     buildStatus = "Approved-UT"
						     sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PbuildStatus='Approved-UT' -PStageName='DevA'"
						     println "Took ${currentBuild.startTimeInMillis}"
                                                     
						      echo 'Exporting application from Dev environment : ' + env.PEGA_DEV
                                                      sh "./gradlew performOperation -Dprpc.service.util.action=export -Dpega.rest.server.url=${devastgs}/PRRestService -Dpega.rest.username=puneeth_export -Dpega.rest.password=rules -Duser.temp.dir=${WORKSPACE}/tmp -Dexport.applicationName=${appname} -Dexport.applicationVersion=${appversion} -Dexport.productName=${productName} -Dexport.productVersion=${productVersion} -Dexport.archiveName='${productName}-${productVersion}-${env.BUILD_NUMBER}.zip' --debug"
						     }catch(err) { // input false
						         echo "This Job has been Aborted"
                                                         currentBuild.result = 'UNSTABLE'
							 buildStatus = 'Aborted-UT'
							 String destinationTestPath = "~/${appname}/${TESTRESULTSFILE}"

							 sh "./gradlew sendUpdateToPega -PtargetURL=${PEGA_DEV} -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PbuildStatus=${buildStatus} -PStageName='DevA'"
							 }
						     }
						      else{
						      sh "ssh pegacoeadm@svl-pgwasda-d1 mkdir -p /var/tmp/CICD/${appname}"
						      sh "exit"
						      sh "scp ${WORKSPACE}/${TESTRESULTSFILE} pegacoeadm@svl-pgwasda-d1:/var/tmp/CICD/${appname}/${TESTRESULTSFILE}"
						      String destinationTestPath = "~/${appname}/${TESTRESULTSFILE}"

						                      //sh "./gradlew sendUpdateToPega -PbuildStatus='Dev_Completed' -PDateFlag=End -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PStageName='DevA'"
								      echo 'Exporting application from Dev environment : ' + env.PEGA_DEV
								      sh "./gradlew performOperation -Dprpc.service.util.action=export -Dpega.rest.server.url=${env.PEGA_DEV}/PRRestService -Dpega.rest.username=puneeth_export -Dpega.rest.password=rules -Duser.temp.dir=${WORKSPACE}/tmp -Dexport.applicationName=${appname} -Dexport.applicationVersion=${appversion} -Dexport.productName='DevOps_Export' -Dexport.productVersion='01-01-01' -Dexport.archiveName='${applicationName}-${applicationVersion}_${buildNumber}.zip' --debug"

                                                         sh "./gradlew sendUpdateToPega -PbuildStatus='Dev_Completed' -PDateFlag=End -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PStageName='DevA'"
								                      }

						     }
						     }
						     }
						     if (module == 'SITA') {
						     sh "./gradlew sendUpdateToPega -PbuildStatus='Sit-Started' -PDateFlag=End -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PStageName='SITA'"
						     echo 'Fetching application archive from Artifactory'
						     sh  "./gradlew fetchFromArtifactory -PartifactoryUser='pega_admin' -PartifactoryPassword='P@ssw0rd_pega' -PproductName=${productName} -PproductVersion=${productVersion}"


                                                      sh "./gradlew performOperation -Dprpc.service.util.action=import -Dpega.rest.server.url=${sitastgs}/PRRestService -Dpega.rest.username=puneeth_export -Dpega.rest.password=rules -Duser.temp.dir=${WORKSPACE}/tmp --debug" 
sh "./gradlew sendUpdateToPega -PbuildStatus='Sit-Completed' -PDateFlag=End -PpegaAppName=${appname} -PpegaAppVersion=${appversion} -PStageName='SITA'"

						     }

				}
                  }
	        }
}	

