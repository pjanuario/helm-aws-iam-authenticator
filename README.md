# kubernetes helm aws-iam-authenticator

This image is based on the official [alpine/helm](https://hub.docker.com/r/alpine/helm/) image and add support for [AWS IAM Authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)

More information follow the instructions of the base image.

# Usage

    # mount local folders in container.
    docker run -ti --rm -v $(pwd):/apps -w /apps \
        -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
        -v ~/.cache/helm:/root/.cache/helm \
        alpine/helm

    # Run helm with special version. The tag is helm's version
    docker run -ti --rm -v $(pwd):/apps -w /apps \
        -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
        -v ~/.cache/helm:/root/.cache/helm \
        alpine/helm:3.1.1

    # run container as command
    alias helm="docker run -ti --rm -v $(pwd):/apps -w /apps \
        -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
        -v ~/.cache/helm:/root/.cache/helm \
        alpine/helm"
    helm --help

    # example in ~/.bash_profile
    alias helm='docker run -e KUBECONFIG="/root/.kube/config:/root/.kube/some-other-context.yaml" -ti --rm -v $(pwd):/apps -w /apps \
        -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
        -v ~/.cache/helm:/root/.cache/helm \
        alpine/helm'

# GitLab Usage

    deploy_integration:
      stage: deploy
      environment:
        name: integration
        url: https://int.myenvironment.com
      variables:
        NAMESPACE: integration
      image:
        name: prnjanuario/helm-aws-iam-auth:latest
        entrypoint: ["/bin/sh", "-c"]
      script:
        - mkdir -p ${KUBEDIR}
        - echo -n ${KUBE_CONFIG} | base64 -d > ${KUBECONFIG}
        - echo -n ${INTEGRATION_CONFIG} | base64 -d > ./integration.yaml
        - helm ls
        - helm upgrade -i --namespace $NAMESPACE -f ./integration.yaml --set image.tag=$CI_COMMIT_SHORT_SHA $CHART_RELEASE $CHART
      only:
        - master
