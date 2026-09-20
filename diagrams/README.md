# C4 diagrams: BP Internet Banking

Editable sources are `.drawio` files (open with draw.io / diagrams.net). PNG exports are in [png/](png/).

| File | C4 diagram type | Audience | What it shows |
| --- | --- | --- | --- |
| [01-context.drawio](01-context.drawio) | System Context (level 1) | Product Owners, business | Customers, staff, the banking system, BP internal systems and external services, without technology detail |
| [02-container.drawio](02-container.drawio) | Containers (level 2) | Technical leaders | Apps, gateway, BFF, microservices, data stores and message topics, with protocols. Logical view only |
| [03-components.drawio](03-components.drawio) | Components (level 3), 8 pages | Engineers, architects | Inside each main container: patterns, protocols, security boundaries |
| [04-deployment.drawio](04-deployment.drawio) | Deployment | Architects, operations, security | Where containers run: edge, primary region, DR region, replication, private links, failover |
| [05-dynamic.drawio](05-dynamic.drawio) | Dynamic, 3 pages | Everyone | Numbered runtime steps: mobile sign-in, onboarding, interbank transfer |

Pages of `03-components.drawio` (one container per page): Transfers and Payments, Movements, Frequent Client, Onboarding, Notification, Audit, Web BFF, API Gateway policy pipeline.

The Code level (level 4) is not drawn; the component level is the deepest level the exercise asks for. A System Landscape diagram is not needed because the design covers a single system.

## C4 practices applied (from Simon Brown's guidance)

- **One level per diagram, no mixed layers.** Container diagrams contain no deployment concepts (no clusters, zones, regions, edge services). All of that lives in the Deployment diagram.
- **Infrastructure is not modelled as containers, with one justified exception.** The API Gateway is shown as a container because the exercise mandates an integration layer with an API Gateway and its policies are part of the design in every environment; the diagram says so. Front Door, WAF, static hosting and clusters appear only in the Deployment diagram.
- **Message bus is not one box.** Each topic and queue is its own container, so publishers and subscribers show real point-to-point dependencies.
- **Component diagrams cover one container each.**
- **Dynamic diagrams** show runtime behaviour with numbered interactions instead of guessing order from the static diagrams.
- **Every element has a name, a type, a technology (where relevant) and a short description; every arrow has a purpose and, where relevant, a protocol.** Each diagram has a title and a legend or key.
- Technology is kept out of the Context diagram.

## Colour and line conventions

- Dark blue: people. Blue: BP systems and containers. Bright blue: Azure managed components. Light blue: components. Grey-violet: external systems.
- Grey solid line: synchronous request. Dashed blue: asynchronous event. Orange: call to an external system or private link. Green: authentication. Purple dashed (Deployment only): replication and failover.

## Assumptions shown in the diagrams

- Web: Angular SPA with a BFF; mobile: Flutter (D2, D3). Back-end language is not fixed; services are shown as REST microservices.
- Each service owns its data store; caches are logical Redis keyspaces per service on one managed cluster (D13).
- Region: South Central US (primary, zone-redundant) with North Central US as warm-standby DR (D1); instance counts are illustrative.
- The Core, Customer Detail system and Authorization Server are drawn as BP systems; in the Deployment diagram they are placed in a BP data center reached by ExpressRoute with a VPN backup. This location is an assumption to confirm.
- The interbank network, civil registry, identity vendor, screening service and SMS/email providers are external and reached only through adapters.
