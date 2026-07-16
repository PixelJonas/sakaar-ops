# components-apps

Reserved for future application workloads, following the same pattern as
`home-ops`'s `components-apps/` — see `docs/adding-apps.md` in that repo
for the file structure convention to follow when the first app is added.
Registering an app also requires adding a `cluster-apps` AppProject and a
`values-apps.yaml` to `bootstrap/`, neither of which exist yet (YAGNI —
add them together with the first real app).
