# Signed policies for a test KeyBrokerService

`policies/kbs-operator.json` is the signed CAS session of the attested e2e test
(`test/e2e/testdata/kbs-sessions/kbs-operator.yaml`), with the MRENCLAVEs of
`registry.scontain.com/cicd/soup-controllers/kbs:poc.1` as the operator starts it (`SCONE_HEAP=2G`,
`SCONE_SYSLIBS=1`). It was signed on 2026-11-04 with the SCONE CLI identity in `~/.cas` that
`make cas-and-policies` uses, and is the session that passed `make test-e2e-kbs-attested`.
`policies/kbs-operator.yaml` is its unsigned source; the operator uploads only the `*.json` files.

The session defines the service `kbs` (so `SCONE_CONFIG_ID=kbs-operator/kbs`). CAS generates the token signing key,
a test CA, the token signing certificate and the HTTPS certificate, and the session injects them with the KBS
configuration, the Rego policy and the admin public key. Its access policy allows `ANY` and it ignores all
advisories: for testing only.

To use it:

1. Push this directory to the test repository, keeping `policies/` at the top: the KeyBrokerService reads every
   `*.json` file directly in `spec.policySource.git.path`.
2. Apply `keybrokerservice.yaml` after replacing the values in `<...>`, with the CAS client and CA bundle Secrets.

The session has no `predecessor`, so CAS accepts it only while it holds no session named `kbs-operator`, for
example a CAS freshly installed by `make cas-and-policies`. To change the session later, add
`predecessor: <hash>` with the hash in `status.cas.sessions` of the KeyBrokerService, sign it again with
`scone session sign`, and commit the new JSON.
