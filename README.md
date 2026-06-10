// global-settings.gradle
beforeSettings { settings ->
	def jFrogURL = 'https://artifactory.jfrog.sbi:443/artifactory/itepaypg-sbiepay2-gradle-virtual/'
	def jFroglabUser = settings.providers.gradleProperty('jfrog_username').getOrNull()
	def jFroglabToken = settings.providers.gradleProperty('jfrog_token').getOrNull()
	def gitlabUser = settings.providers.gradleProperty('gitlab_username').getOrNull()
	def gitToken = settings.providers.gradleProperty('gitlab_token').getOrNull()
	def gitSDKToken = settings.providers.gradleProperty('gitlab_sdk_token').getOrNull()
	println "jFrogURL: ${jFrogURL}"
	println "jFroglabUser: ${jFroglabUser}"
	println "jFroglabToken: ${jFroglabToken}"
	println "gitlabUser: ${gitlabUser}"
	println "gitToken: ${gitToken}"
	println "gitSDKToken: ${gitSDKToken}"

    settings.pluginManagement {
        repositories {
            maven {
				url jFrogURL
				credentials {
					username = jFroglabUser
					password = jFroglabToken
				}
			}
        }
    }

    settings.dependencyResolutionManagement {
        // Enforce these repositories for all projects
        repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
        repositories {
			maven {
				url 'https://artifactory.jfrog.sbi/ui/native/itepaypg-sbiepay2-maven-local/'
				credentials {
					username = jFroglabUser
					password = jFroglabToken
				}
			}
             maven {
				url 'https://artifactory.jfrog.sbi:443/artifactory/itepaypg-sbiepay2-gradle-virtual/'
				credentials {
					username = jFroglabUser
					password = jFroglabToken
				}
			}
			maven {//SBI JAVA Utilities Package Registry to download utilities jar.
				url "https://gitlab.epay.sbi/api/v4/projects/16/packages/maven"
				credentials(PasswordCredentials) {
					username = gitlabUser
					password = gitToken
				}
				authentication {
					basic(BasicAuthentication)
				}
			}
			maven {//SBI JAVA SDK Package Registry to download in development java SDK.
				url 'https://gitlab.epay.sbi/api/v4/projects/8/packages/maven'
				credentials(PasswordCredentials) {
					username = gitlabUser
					password = gitSDKToken
				}
				authentication {
					basic(BasicAuthentication)
				}
			}
			flatDir {
				dirs "libs"
			}
        }
    }
}
