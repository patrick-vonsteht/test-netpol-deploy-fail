This repository contains a minimal example of an issue in which helm creates orphaned resources.

# Reproduction Case 1

In this test case, we first successfully deploy a helm chart without a network policy. Then we add a network policy, but
simulate that the deployment fails. Afterwards we rename the network policy. As a result we end up with two network
policies, although there should only be one. The first network policy seems to be orphaned, as it isn't deleted when 
uninstalling the helm chart:

```
PS > helm upgrade --install --wait --timeout 5s test1-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=test --set netpolPort=80 --set netpolEnabled=false --set fail=false
Release "test1-netpol-deploy-fail" does not exist. Installing it now.
NAME: test1-netpol-deploy-fail
LAST DEPLOYED: Thu Jan 19 13:04:59 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

PS > helm upgrade --install --wait --timeout 5s test1-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=test --set netpolPort=80 --set netpolEnabled=false --set fail=true^C

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-6f57dc577tc7td   1/1     Running   0          15s

PS > kubectl get networkpolicies
No resources found in default namespace.

PS > helm upgrade --install --wait --timeout 5s test1-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=test --set netpolPort=80 --set netpolEnabled=true --set fail=true
Error: UPGRADE FAILED: timed out waiting for the condition

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-6f57dc577tc7td   1/1     Running   0          36s
test1-netpol-deploy-fail-test-netpol-deploy-fail-76cf946d8lgmtf   0/1     Running   0          9s

PS > kubectl get networkpolicies
NAME                                                    POD-SELECTOR                                                                                         AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-test   app.kubernetes.io/instance=test1-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   14s

PS > helm upgrade --install --wait --timeout 5s test1-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=abc --set netpolPort=80 --set netpolEnabled=true --set fail=true
Error: UPGRADE FAILED: timed out waiting for the condition

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-6f57dc577tc7td   1/1     Running   0          71s
test1-netpol-deploy-fail-test-netpol-deploy-fail-76cf946d8lgmtf   0/1     Running   0          44s

PS > kubectl get networkpolicies
NAME                                                    POD-SELECTOR                                                                                         AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-abc    app.kubernetes.io/instance=test1-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   16s
test1-netpol-deploy-fail-test-netpol-deploy-fail-test   app.kubernetes.io/instance=test1-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   53s

PS > helm uninstall test1-netpol-deploy-fail
release "test1-netpol-deploy-fail" uninstalled

PS > kubectl get pods
No resources found in default namespace.

PS > kubectl get networkpolicies
NAME                                                    POD-SELECTOR                                                                                         AGE
test1-netpol-deploy-fail-test-netpol-deploy-fail-test   app.kubernetes.io/instance=test1-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   7m55s
PS >


```

# Reproduction Case 2

In this test case, we first successfully deploy a helm chart with a network policy. Then we rename the network policy
and change the port that it applies to and simulate that this deployment fails. Afterwards we rename the network policy
again. As a result we end up with two network policies, although there should only be one. The first network policy 
seems to be orphaned, as it isn't deleted when uninstalling the helm chart:

```
PS > helm upgrade --install --wait --timeout 5s test2-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=test --set netpolPort=80 --set netpolEnabled=true --set fail=false
Release "test2-netpol-deploy-fail" does not exist. Installing it now.
NAME: test2-netpol-deploy-fail
LAST DEPLOYED: Thu Jan 19 13:15:43 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-879fc689bss2nk   1/1     Running   0          8s

PS > kubectl get networkpolicies
NAME                                                    POD-SELECTOR                                                                                         AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-test   app.kubernetes.io/instance=test2-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   12s

PS > helm upgrade --install --wait --timeout 5s test2-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=abc --set netpolPort=70 --set netpolEnabled=true --set fail=true
Error: UPGRADE FAILED: timed out waiting for the condition

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-879fc689bss2nk   1/1     Running   0          41s
test2-netpol-deploy-fail-test-netpol-deploy-fail-88df8f6c79qmkk   0/1     Running   0          7s

PS > kubectl get networkpolicies
NAME                                                   POD-SELECTOR                                                                                         AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-abc   app.kubernetes.io/instance=test2-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   12s

PS > helm upgrade --install --wait --timeout 5s test2-netpol-deploy-fail .\test-netpol-deploy-fail\ --set netpolNameSuffix=123 --set netpolPort=70 --set netpolEnabled=true --set fail=true
Error: UPGRADE FAILED: timed out waiting for the condition

PS > kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-879fc689bss2nk   1/1     Running   0          73s
test2-netpol-deploy-fail-test-netpol-deploy-fail-88df8f6c79qmkk   0/1     Running   0          39s

PS > kubectl get networkpolicies
NAME                                                   POD-SELECTOR                                                                                         AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-123   app.kubernetes.io/instance=test2-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   9s
test2-netpol-deploy-fail-test-netpol-deploy-fail-abc   app.kubernetes.io/instance=test2-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   41s

PS > helm uninstall test2-netpol-deploy-fail
release "test2-netpol-deploy-fail" uninstalled

PS > kubectl get pods
No resources found in default namespace.

PS > kubectl get networkpolicies
NAME                                                   POD-SELECTOR                                                                                         AGE
test2-netpol-deploy-fail-test-netpol-deploy-fail-abc   app.kubernetes.io/instance=test2-netpol-deploy-fail,app.kubernetes.io/name=test-netpol-deploy-fail   57s
```