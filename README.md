1. Install:
`$ kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/definitions/release/install.yaml`

---

2. Apply policies:
```
# Mutation: add a label domain=ng to resources
$ kubectl apply -f add_labels.yaml`
$ k get ClusterPolicy
NAME         BACKGROUND   ACTION
add-labels   true         audit

# Generator: Resource quota per namespace
$ k create -f ns_res_quota.yaml
clusterpolicy.kyverno.io/add-ns-quota created
$ k get clusterpolicy
NAME           BACKGROUND   ACTION
add-labels     true         audit
add-ns-quota   true         audit

# Validators
$ k create -f deny_latest_tag.yaml -f deny_priv_esc.yaml -f deny_runas_root.yaml
clusterpolicy.kyverno.io/disallow-latest-tag created
clusterpolicy.kyverno.io/deny-privilege-escalation created
clusterpolicy.kyverno.io/require-run-as-non-root created
$ k get clusterpolicy
NAME                        BACKGROUND   ACTION
add-labels                  true         audit
add-ns-quota                true         audit
deny-privilege-escalation   true         audit
disallow-latest-tag         true         audit
require-run-as-non-root     true         audit

```

---

3. Test
```
❯ k get clusterpolicy | grep enforce
multi-tenancy               false        enforce
restrict-nodeport           false        enforce
❯ k delete pod nginx
Error from server: admission webhook "validate.kyverno.svc" denied the request:

resource Pod/default/nginx was blocked due to the following policies

multi-tenancy:
  block-delete-for-resources: Deleting Pod/nginx is not allowed
❯ k expose pod nginx --port=80 --type=NodePort
Error from server: admission webhook "validate.kyverno.svc" denied the request:

resource Service/default/nginx was blocked due to the following policies

restrict-nodeport:
  validate-nodeport: 'validation error: Services of type NodePort are not allowed.
    Rule validate-nodeport failed at path /spec/type/'
```

