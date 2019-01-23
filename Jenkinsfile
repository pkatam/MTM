/* 
* Copyright (c) 2019 and Confidential to Carefirst All rights reserved.  
*/ 

import groovy.json.*
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;

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
	    def EnvList = []
	    String a = "${env.JOB_NAME}".split('/')[0] as String
	    String AppPath= "/home/pegacoeadm/" + "${a}"
	    println "$AppPath"
	    //def inputFile = new File("/home/pegacoeadm/test1.json")
	    def inputFile = new File("${AppPath}")
	    def InputJSON = new JsonSlurperClassic().parseFile(inputFile, 'UTF-8')
            def stgs,devbstgs,devastgs,sitastgs,sitbstgs,uatstgs,ttstgs,prodstgs
	    InputJSON.AppDetails.each { 
	    stgs=it."$item";
	    devbstgs=it."$devb";
	    devastgs=it."$deva";
	    sitastgs=it."$sita";
	    sitbstgs=it."$sitb";
	    uatstgs=it."$uat";
            ttstgs=it."$tt";
	    prodstgs=it."$prod";
	    println "${devastgs}";
	    if (devastgs) { 
	    
	    	//EnvList.add("${devastgs}")
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
			                           echo 'Initiating UT...'
						   withEnv(['TESTRESULTSFILE="TestResult.xml"']) {
						   sh "./gradlew executePegaUnitTests -PtargetURL=${PEGA_DEV} -PpegaUsername=puneeth_export -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE}"
						   junit '**/TestResult.xml'
						   script {

						    if (currentBuild.result != null) {
						     input(message: 'Unit Tests have failed, would you like to abort the pipeline?')
						     }
						     }
						     }
						     }
				}
                  }
	        }
				
}	

