# Breaking up Is Hard, but the Future Is with Helm 4

Matt Farina and Joe Julian · KubeCon + CloudNativeCon North America · 10 November 2026 · 3:40–4:10 pm MST.

22 spoken slides. Aim to reserve the last five minutes of the 30-minute session for Q&A. Actual runtime needs rehearsal. Slide 23 is a reference appendix.

## 01. Title

Welcome. Matt and Joe introduce the talk. Helm 3 carried the ecosystem for six years. The purpose today is to explain what Helm 4 changes, why the upgrade path is manageable, and where contributors can help shape the future.

Sources:

- https://kubecon-cloudnativecon-north-america-2026.sessionize.com/session/1289638

## 02. Who we are

Briefly introduce yourselves and your relationship to Helm. The audience should understand that this is a maintainer perspective grounded in the project and its wider community.

Sources:

- https://kubecon-cloudnativecon-north-america-2026.sessionize.com/session/1289638

## 03. What are we breaking up with?

Move from the title into the reason for a major release. The target is architectural constraints, while keeping existing charts and releases viable.

## 04. What Helm still is

Start with the durable job of Helm: it is the package manager for Kubernetes. A chart is a versioned package of Kubernetes resource definitions. Helm installs a chart to create a release, and it manages that release through upgrades and rollback. A chart can create more than one separately tracked release. Helm 4 changes parts of the implementation and some integrations, but this core model remains recognizable.

Sources:

- https://helm.sh/docs/intro/introduction/

## 05. Developers rate Helm as mature

Helm 3 was released in November 2019. In the CNCF Q1 2026 Technology Radar, 94% of surveyed developers familiar with Helm rated its maturity 4 or 5 stars, the strongest maturity result among application delivery tools. The report defines maturity in terms of stability and reliability. This is a perception survey about Helm overall, not an uptime measurement or a Helm 3-only result. The broad installed base and integrations made compatibility important as Helm 4 was designed.

Sources:

- https://www.cncf.io/wp-content/uploads/2026/03/Q1-2026-CNCF-Technology-Radar-Report.pdf
- https://helm.sh/blog/the-road-to-helm-4/

## 06. The cost of that compatibility promise

Helm 3 accumulated technical debt and architectural limits. The project could continue adding compatible features, but some improvements required changes to public SDK APIs and package structure. A major version provided a clear boundary for those changes. The point is not to fault Helm 3; its compatibility rules did their job for a very broad user base.

Sources:

- https://helm.sh/blog/helm-v3-end-of-life/
- https://helm.sh/docs/overview/

## 07. Most of your investment stays

Most teams can adopt Helm 4 without a chart rewrite or release migration. Existing Helm 3 charts should deploy with Helm 4, and existing Helm 3 releases can generally be upgraded by Helm 4. State the qualification: test your actual workflows, because plugins, post-renderers, authentication, and SDK consumers may need attention.

Sources:

- https://helm.sh/blog/helm-v3-end-of-life/
- https://helm.sh/docs/overview/

## 08. The Helm 3 support clock

The timeline changed after the original Helm 4 announcement. Helm 4.0 shipped in November 2025. Helm 3 had a final limited minor on September 9, 2026. Security fixes end on February 10, 2027. After that, no Helm 3 security patches or Kubernetes client updates are expected. Encourage the audience to use the support window deliberately.

Sources:

- https://helm.sh/blog/helm-v3-end-of-life/

## 09. What changed in Helm 4?

Transition from motivation into the capabilities available today.

## 10. Four foundations for the next chapter

Preview the next section. Extensibility, operations, distribution, and internal architecture are the four themes. The examples on the next slides are representative changes, not an exhaustive changelog.

Sources:

- https://helm.sh/docs/overview/

## 11. Plugins have a clearer shape

Helm 4 has three explicit plugin types: CLI, getter, and post-renderer. The redesigned system allows future extension types, and an optional WebAssembly runtime provides a more controlled execution environment. Existing Helm 3 plugins continue to work. Plugin maintainers can adopt the versioned plugin schema over time.

Sources:

- https://helm.sh/docs/overview/
- https://helm.sh/docs/plugins/overview/
- https://helm.sh/docs/plugins/migrate/

## 12. A few integrations do change

In Helm 3, --post-renderer accepts an executable path. In Helm 4 it accepts a plugin name. Workflows that pass a path need to package that executable as a postrenderer/v1 plugin and invoke the name. Also, helm registry login now accepts a domain name only, not a full URL. Ask the audience to search their CI and deployment scripts for both.

Sources:

- https://helm.sh/docs/overview/
- https://helm.sh/docs/plugins/migrate/
- https://helm.sh/blog/helm-v3-end-of-life/

## 13. Apply behavior remembers the release

New Helm 4 installs use server-side apply by default. When Helm 4 upgrades or rolls back an existing Helm 3 release, it keeps that release on client-side apply by default. This latching behavior reduces surprises during a migration. Teams can override explicitly with --server-side. Test workloads where controllers and other tools touch the same Kubernetes objects.

Sources:

- https://helm.sh/docs/overview/

## 14. Better signals while you wait

Helm 4 integrates kstatus for resource watching. The goal is more informative waiting and deployment status for complex applications. Invite users to test multi-resource workloads and report cases where the status is inaccurate or hard to interpret.

Sources:

- https://helm.sh/docs/overview/

## 15. Pin the chart you meant to install

Helm 4 supports installing OCI charts by digest. That lets a deployment request an exact artifact and reject a mismatch. The release also adds content-based chart caching and reproducible archive builds. These are useful supply-chain building blocks, though they do not alone solve every provenance or policy problem.

Sources:

- https://helm.sh/docs/overview/
- https://helm.sh/blog/helm-4-released/

## 16. Small changes you will notice

Multi-document values make it easier to organize configuration. Errors are clearer. Two common flags have clearer names: --atomic becomes --rollback-on-failure, and --force becomes --force-replace. The old names still work in Helm 4 but warn, so update scripts intentionally.

Sources:

- https://helm.sh/docs/overview/

## 17. A foundation for the next chart format

Helm 4 restructures its SDK APIs and supports multiple chart API versions. Charts v3 is early and experimental. Chart API v2 continues to work, so moving to Helm 4 does not mean migrating charts to v3. Developers can test the experiment and share what capabilities a future chart format should provide.

Sources:

- https://helm.sh/docs/overview/

## 18. How do we move forward?

Transition into migration steps and community action.

## 19. Upgrade in five deliberate steps

The Helm migration checklist is straightforward: test representative charts and existing releases; run the exact commands in CI and deployment automation; inspect plugins and especially path-based post-renderers; verify OCI authentication and installs; then roll out the Helm 4 binary. The best migration pilot uses a real application rather than a toy chart.

Sources:

- https://helm.sh/blog/helm-v3-end-of-life/
- https://helm.sh/docs/overview/

## 20. Make the first upgrade boring

A sample runbook: inventory releases; lint and render a representative chart; install a canary in a test namespace; inspect its status; then exercise an actual upgrade of a Helm 3 release. Test a controlled failure and rollback path as well. Commands on this slide are illustrative and assume a namespace and credentials already exist.

Sources:

- https://helm.sh/docs/overview/
- https://helm.sh/blog/helm-v3-end-of-life/

## 21. There is a way in for you

Invite specific participation: test a workflow and report findings, join #helm-dev or the public development meeting, and contribute to a project matching your strengths. Documentation, tests, and support matter alongside Go code. Point everyone to helm.sh/community for current meeting details.

Sources:

- https://helm.sh/community/
- https://helm.sh/community/communication

## 22. The future is with Helm 4

Close on three takeaways. First, most charts and releases continue across the version boundary. Second, test the workflow around Helm, especially plugins, CI flags, and OCI authentication. Third, Helm 4 is a foundation and the community can shape what comes next. Invite questions. Aim to reserve five minutes for Q&A; rehearsal will determine the actual running time.

Sources:

- https://helm.sh/blog/helm-v3-end-of-life/
- https://helm.sh/docs/overview/

## 23. Sources and further reading

Reference slide. Support timeline is based on the updated June 2026 announcement. Recheck release notes immediately before the event for any late changes.

Sources:

- https://helm.sh/docs/overview/
- https://helm.sh/blog/helm-v3-end-of-life/
- https://helm.sh/docs/plugins/migrate/
- https://helm.sh/community/
