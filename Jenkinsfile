podTemplate(
    label: "mybuildpod", 
    cloud: "openshift", 
    serviceAccount: "jenkins",
    containers: [
        containerTemplate(
            name: "jnlp", 
            image: "registry.redhat.io/openshift4/ose-jenkins-agent-base", 
            resourceRequestMemory: "512Mi", 
            resourceLimitMemory: "512Mi", 
            envVars: [
                envVar(key: "CONTAINER_HEAP_PERCENT", value: "0.25") 
            ]
        ),
        containerTemplate(
            name: "git", 
            image: "nexus.ocp.icp.true-north.hr/repository/myrepo/build-git",
            command: 'cat',
            ttyEnabled: true
        ),
        containerTemplate(
            name: "s2i", 
            image: "nexus.ocp.icp.true-north.hr/repository/myrepo/build-s2i",
            command: 'cat',
            ttyEnabled: true
        ),
        containerTemplate(
            name: "oc", 
            image: "nexus.ocp.icp.true-north.hr/repository/myrepo/build-oc",
            command: 'cat',
            ttyEnabled: true
        ),
        containerTemplate(
            name: "buildah", 
            image: "nexus.ocp.icp.true-north.hr/repository/myrepo/build-buildah",
            command: 'cat',
            ttyEnabled: true,
            privileged: true,
            envVars: [envVar(key: 'REGISTRY_AUTH_FILE', value: '/home/jenkins/ibm-pull-secret/.dockerconfigjson')]
        )
    ],
    volumes: [
        emptyDirVolume(mountPath: '/var/lib/containers'),
        secretVolume(mountPath: '/home/jenkins/ibm-pull-secret', secretName: 'ibm-pull-secret')
    ]
) 
{
    node("mybuildpod") {
        stage("Checkout") {
		checkout scm
        }
        stage("Generate") {
            container("s2i") {
                sh "s2i build openshift-quickstarts/undertow-servlet registry.redhat.io/redhat-openjdk-18/openjdk18-openshift --as-dockerfile gen/Dockerfile --image-scripts-url image:///usr/local/s2i"
            }
        }
        stage("Build") {
            container("buildah") {
                sh "cd gen; buildah --storage-driver=vfs bud -t nexus.ocp.icp.true-north.hr/repository/myrepo/myjbossapp .;"
                sh "buildah --storage-driver=vfs push nexus.ocp.icp.true-north.hr/repository/myrepo/myjbossapp"
            }
        }
        stage("Deploy") {
            container("git") {
                sh "git clone https://github.com/true-north-engineering/ocp-lab"
            }
            container("oc") {
                openshift.withCluster() {
                    openshift.withProject("myjbossapp") {
                        openshift.apply("-f", "ocp-lab/Day07/examples/deployment.yaml");
                    }
                }
            }
        }
    }
}

