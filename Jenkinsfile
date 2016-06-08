stage "collect build info"
def packageName = "github.com/opencontainers/runc"
def buildTags

wrappedNode(label: "ubuntu") {
  deleteDir()
  checkout scm
  buildTags = getOutput("make print-BUILDTAGS")
}

stage "validate and test"
golangTester(
  label: "ubuntu && docker",
  package: packageName,
  go_version: "1.6.2",
  gocov_args: "-tags \"${buildTags}\"",
  max_warnings: 0
)()

wrappedNode(label: "ubuntu") {
  withChownWorkspace {
    deleteDir()
    checkout scm
    timeout(45) {
      stage "build image"

      def imageId = "dockerbuildbot/runc-test:${gitCommit()}"
      def image
      try {
        stage "build test image"
        sh "docker build -t ${imageId} -f script/test_Dockerfile ."
        image = docker.image(imageId)

        stage "integration test"
        sh """#!/bin/bash
          docker run \\
            --rm \\
            -it \\
            --privileged \\
            -v \$(pwd):/go/src/${packageName} \\
            -v \$(pwd)/results:/output \\
            ${image.id} \\
            make localintegration | tail -n +2 | tee /output/integration.tap
        """
        step([$class: "TapPublisher", testResults: "results/integration.tap"])
      } finally {
        if (image) { sh "docker rmi ${image.id} ||:" }
      }
    }
  }
}
