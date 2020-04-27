pipeline {
    agent {label 'master'} 
	environment { 
		
		// Custom Variables
		// slaveLabel = "SOFCM3"
		// Fixed Parameters - *** DO NOT CHANGE ***
		buildNumber = "${BUILD_ID}"
		buildPackage = "Fusion-${env.buildNumber}-project.zip"
		mavenRepository = "D:\\Maven\\.m2\\repository\\com\\sse\\retail\\Fusion"
		stageDir = "BuildPackages\\${env.buildNumber}"
		targetStageDir = "BuildPackages/${env.buildNumber}"
		Build_Package = "/products/oracle/stage/release/${env.targetStageDir}/Fusion-${env.buildNumber}"
	}
    stages {
			// Print values of all the environment variables
            stage ('Validate Environment Variables'){
                agent {label 'master'}
                steps{
				    // send email notification
					wrap([$class: 'BuildUser']) {
                    emailext (
                        subject: "DEPLOYMENT STARTED: Job '${env.JOB_NAME} [DEPLOYMENT Number - ${env.BUILD_NUMBER}]'",
                        body: """<p>DEPLOYMENT STARTED: Job '${env.JOB_NAME} started with below details':</p>
                        <p>Build Number - ${env.buildNumber}</p>
						<p>Environment - ${slaveLabel}</p>
                        <p>Deployment Owner - ${BUILD_USER}</p>
                        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                        to: "SB14762@sse.com, VY23150@sse.com"
                    )
				    echo 'Validate Environment Variables'
					echo "${slaveLabel}"
					echo "${env.buildNumber}"
					echo "${env.buildPackage}"
					echo "${env.mavenRepository}"
					echo "${env.stageDir}"
					echo "${env.Build_Package}"
					}
				}
			}
			
			// Get Build Package from Maven
            stage ('Download Build Package'){
                agent {label 'master'}
				steps{
				    echo 'This is Download Build Package'
					bat """
					mkdir ${env.stageDir}
					xcopy ${env.mavenRepository}\\${env.buildNumber}\\${env.buildPackage} ${env.stageDir} /R /Y
					"""
					stash name: "build-stash", includes: "${env.stageDir}/*.zip"
				}
			}
			
			// Transfer Build Package from Master
            stage ('Transfer Build Package'){
                agent {label "${slaveLabel}"}
				steps{
				    echo 'This is Transfer Build Package'
					dir("/products/oracle/stage/release"){
					unstash "build-stash"
					}
				}
            }
			
			// Extract Build Package from Master
            stage ('Extract Build Package'){
                agent {label "${slaveLabel}"}
				steps{
				    sh """
					echo 'This is Extract Build Package'
					cd /products/oracle/stage/release/${env.targetStageDir}/
					unzip -o ${env.buildPackage}
					chmod -R 755 ${env.Build_Package}/*
					"""
					}
				}
			
			stage ('Deployment'){
				parallel{
					stage ('SOA'){
						agent {label "${slaveLabel}"}
						stages {
						
							// Export SOA MDS
							stage ('Export SOA MDS'){
								steps{
									echo 'This is Export SOA MDS'
									sh """
									cd ${env.Build_Package}/Scripts
									./sse_stp_export_soa_mds.sh ExportedMDS_\$(date '+%d%m%Y_%H%M%S').zip
									"""
								}
							}
							
							// Tokenise SOA MDS
							stage ('Tokenise SOA MDS'){
								steps{
									echo 'This is Tokenise SOA MDS'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA tokenisation_mds ${env.Build_Package}
									"""
								}
							}
							
							// Tokenise SOA Composites of 'default' partition
							stage ('Tokenise SOA Composite - default'){
								steps{
									echo 'This is Tokenise SOA Composite - default'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA tokenisation_soa_default ${env.Build_Package}
									"""
								}
							}
							
							// Tokenise SOA Composites of 'DCCi' partition
							stage ('Tokenise SOA Composite - DCCi'){
								steps{
									echo 'This is Tokenise SOA Composite - DCCi'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA tokenisation_soa_DCCI ${env.Build_Package}
									"""
								}
							}
							
							// Tokenise SOA Composites of 'CCB2' partition
							stage ('Tokenise SOA Composite - CCB2'){
								steps{
									echo 'This is Tokenise SOA Composite - CCB2'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA tokenisation_soa_CCB2 ${env.Build_Package}
									"""
								}
							}
							
							// Deploy SOA MDS
							stage ('Deploy SOA MDS'){
								steps{
									echo 'This is Deploy SOA MDS'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA mds ${env.Build_Package}
									cd ${env.Build_Package}/Scripts
									./sse_stp_export_soa_mds.sh ExportedMDS_\$(date '+%d%m%Y_%H%M%S').zip
									"""
								}
							}
							
							// Deploy SOA Composites of 'default' partition
							stage ('Deploy SOA Composite - default'){
								steps{
									echo 'This is Deploy SOA Composite - default'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA composite_default ${env.Build_Package}
									"""
								}
							}
							
							// Deploy SOA Composites of 'DCCi' partition
							stage ('Deploy SOA Composite - DCCi'){
								steps{
									echo 'This is Deploy SOA Composite - DCCi'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA composite_DCCI ${env.Build_Package}
									"""
								}
							}
							
							// Deploy SOA Composites of 'CCB2' partition
							stage ('Deploy SOA Composite - CCB2'){
								steps{
									echo 'This is Deploy SOA Composite - CCB2'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh SOA composite_CCB2 ${env.Build_Package}
									"""
								}
							}
						
						}
					}
					
					stage ('OSB'){
						agent {label "${slaveLabel}"}
						stages{							
							// Tokenise OSB Projects
							stage ('Tokenise OSB Projects'){
								steps{
									echo 'This is Tokenise OSB Projects'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh OSB tokenise_osb ${env.Build_Package}
									"""
								}
							}
							
							// Build OSB Projects
							stage ('Build OSB Projects'){
								steps{
									echo 'This is Build OSB Projects'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh OSB build_osb ${env.Build_Package}
									"""
								}
							}
							
							// Deploy OSB Projects
							stage ('Deploy OSB Projects'){
								steps{
									echo 'This is Deploy OSB Projects'
									sh """
									cd ${env.Build_Package}/Scripts
									sh ./deploy_mtp_fusion.sh OSB deploy_osb ${env.Build_Package}
									"""
								}
							}
						}
					}
					
				}
            }
			
			// Clear deployed Build Package
			stage ('Clear Build Package'){
				agent {label "${slaveLabel}"}
				steps{
					sh """
					echo 'This is Clear Build Package'
					rm -rf /products/oracle/stage/release/${env.targetStageDir}/
					"""
				}
			}
        }
        
	
	// Send completion or failure email notification
	post {
	    success {
			wrap([$class: 'BuildUser']) {
            emailext (
                subject: "DEPLOYMENT SUCCESSFUL: Job '${env.JOB_NAME} [DEPLOYMENT Number - ${env.BUILD_NUMBER}]'",
                body: """<p>DEPLOYMENT SUCCESSFUL: Job '${env.JOB_NAME} [DEPLOYMENT Number - ${env.BUILD_NUMBER}]':</p>
                <p>Build Number - ${env.buildNumber}</p>
				<p>Environment - ${slaveLabel}</p>
                <p>Deployment Owner - ${BUILD_USER}</p>
				<p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                to: "uttam.shukla72@gmail.com"
            )
			}
	    }
	    
        failure {
            wrap([$class: 'BuildUser']) {
			emailext (
                attachLog: true,
				subject: "DEPLOYMENT FAILED: Job '${env.JOB_NAME} [DEPLOYMENT Number - ${env.BUILD_NUMBER}]'",
                body: """<p>DEPLOYMENT FAILED: Job '${env.JOB_NAME} [DEPLOYMENT Number - ${env.BUILD_NUMBER}]':</p>
                <p>Build Number - ${env.buildNumber}</p>
				<p>Environment - ${slaveLabel}</p>
                <p>Deployment Owner - ${BUILD_USER}</p>
				<p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                to: "uttam.shukla72@gmail.com"
            )
			}
        }
    }
}   
