Certainly! Terraform provides a variety of functions and looping constructs that can be used to dynamically generate resources and configurations. Below are some examples to help you get started with complex Terraform loops and functions in local configurations.

### Example 1: Using `for_each` with a Map

The `for_each` meta-argument allows you to create multiple instances of a resource based on a map or set.

```hcl
locals {
  servers = {
    "web1" = {
      ip   = "10.0.1.10"
      port = 80
    }
    "web2" = {
      ip   = "10.0.1.11"
      port = 80
    }
  }
}

resource "aws_instance" "web" {
  for_each = local.servers

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = each.key
  }
}

output "server_ips" {
  value = {
    for k, v in aws_instance.web : k => v.private_ip
  }
}
```

### Example 2: Using `count` with a List

The `count` meta-argument allows you to create multiple instances of a resource based on a list.

```hcl
locals {
  server_names = ["web1", "web2", "web3"]
}

resource "aws_instance" "web" {
  count = length(local.server_names)

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = local.server_names[count.index]
  }
}

output "server_ips" {
  value = aws_instance.web[*].private_ip
}
```

### Example 3: Using `for` Expressions in Outputs

You can use `for` expressions in outputs to transform and format data.

```hcl
locals {
  servers = {
    "web1" = {
      ip   = "10.0.1.10"
      port = 80
    }
    "web2" = {
      ip   = "10.0.1.11"
      port = 80
    }
  }
}

output "server_urls" {
  value = [
    for name, details in local.servers : "${name}.example.com:${details.port}"
  ]
}
```

### Example 4: Using Functions to Manipulate Data

Terraform provides a variety of built-in functions that can be used to manipulate data.

```hcl
locals {
  server_ips = ["10.0.1.10", "10.0.1.11", "10.0.1.12"]
  subnet_mask = cidrnetmask("10.0.1.0/24")
}

output "formatted_ips" {
  value = formatlist("%s/%s", local.server_ips, local.subnet_mask)
}
```

### Example 5: Using `dynamic` Blocks

`dynamic` blocks can be used to generate nested configuration blocks dynamically.

```hcl
locals {
  security_groups = {
    "web" = ["22", "80", "443"]
    "db"  = ["3306"]
  }
}

resource "aws_security_group" "example" {
  for_each = local.security_groups

  name = each.key

  dynamic "ingress" {
    for_each = each.value

    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

These examples should give you a good starting point for working with complex Terraform loops and functions in local configurations. Feel free to experiment and adapt these examples to fit your specific use cases.

Certainly! Let's dive into a more complex example that combines multiple Terraform features, including `for_each`, `dynamic` blocks, functions, and local values. This example will create a set of AWS security groups with varying rules based on a complex data structure.

### Complex Example: Dynamic Security Groups with Custom Rules

```hcl
locals {
  services = {
    "web" = {
      description = "Web Service Security Group"
      rules = {
        "http" = {
          from_port   = 80
          to_port     = 80
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
        "https" = {
          from_port   = 443
          to_port     = 443
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
        "ssh" = {
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["10.0.0.0/8"]
        }
      }
    }
    "db" = {
      description = "Database Service Security Group"
      rules = {
        "mysql" = {
          from_port   = 3306
          to_port     = 3306
          protocol    = "tcp"
          cidr_blocks = ["10.0.0.0/8"]
        }
        "ssh" = {
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["10.0.0.0/8"]
        }
      }
    }
  }
}

resource "aws_security_group" "service" {
  for_each = local.services

  name        = each.key
  description = each.value.description

  dynamic "ingress" {
    for_each = each.value.rules

    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

output "security_group_ids" {
  value = {
    for k, v in aws_security_group.service : k => v.id
  }
}

output "security_group_rules" {
  value = {
    for k, v in aws_security_group.service : k => [
      for rule in v.ingress : "${rule.from_port}-${rule.to_port}/${rule.protocol}"
    ]
  }
}
```

### Explanation:

1. **Local Values (`locals`)**:
   - Defines a complex data structure `services` containing security group configurations for different services (e.g., "web", "db").
   - Each service has a description and a set of rules (e.g., "http", "https", "ssh", "mysql").

2. **Resource (`aws_security_group`)**:
   - Uses `for_each` to iterate over the `services` map.
   - Creates a security group for each service with a name and description.
   - Uses `dynamic "ingress"` to dynamically generate ingress rules based on the rules defined in the `services` map.

3. **Outputs**:
   - `security_group_ids`: Outputs the IDs of the created security groups.
   - `security_group_rules`: Outputs the rules of each security group in a formatted string.

This example demonstrates how to use Terraform's advanced features to create dynamic and complex infrastructure configurations. You can further extend this example by adding more services, rules, or custom logic as needed.

Certainly! Let's create five complex examples that showcase the power of Terraform's local values, functions, and output transformations. Each example will be designed to demonstrate a different aspect of Terraform's capabilities.

### Example 1: Complex Data Transformation with Functions

```hcl
locals {
  raw_data = {
    "server1" = {
      ip     = "10.0.1.10"
      memory = "16GB"
      cpu    = "4 cores"
    }
    "server2" = {
      ip     = "10.0.1.11"
      memory = "32GB"
      cpu    = "8 cores"
    }
  }

  transformed_data = {
    for k, v in local.raw_data : k => {
      ip     = v.ip
      memory = tonumber(replace(v.memory, "/GB/", ""))
      cpu    = tonumber(replace(v.cpu, "/ cores/", ""))
    }
  }
}

output "transformed_data" {
  value = local.transformed_data
}
```

### Example 2: Dynamic Resource Creation with Nested Loops

```hcl
locals {
  environments = {
    "dev" = {
      region = "us-west-1"
      instances = {
        "web" = { type = "t2.micro", count = 2 }
        "db"  = { type = "t2.medium", count = 1 }
      }
    }
    "prod" = {
      region = "us-east-1"
      instances = {
        "web" = { type = "t2.small", count = 4 }
        "db"  = { type = "t2.large", count = 2 }
      }
    }
  }
}

resource "aws_instance" "example" {
  for_each = {
    for env, details in local.environments : env => {
      for name, config in details.instances : "${env}-${name}" => config
    }
  }

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value.type
  count         = each.value.count

  tags = {
    Name = each.key
  }
}

output "instance_counts" {
  value = {
    for k, v in aws_instance.example : k => length(v)
  }
}
```

### Example 3: Advanced Output Transformation with Functions

```hcl
locals {
  logs = [
    { timestamp = "2023-01-01T00:00:00Z", message = "Starting service" },
    { timestamp = "2023-01-01T00:01:00Z", message = "Service started" },
    { timestamp = "2023-01-01T00:02:00Z", message = "Error: connection lost" },
  ]

  formatted_logs = [
    for log in local.logs : "${log.timestamp} - ${log.message}"
  ]
}

output "formatted_logs" {
  value = join("\n", local.formatted_logs)
}
```

### Example 4: Complex Data Filtering and Mapping

```hcl
locals {
  users = [
    { name = "Alice", age = 25, active = true },
    { name = "Bob", age = 30, active = false },
    { name = "Charlie", age = 35, active = true },
  ]

  active_users = [
    for user in local.users : user if user.active
  ]

  user_emails = {
    for user in local.active_users : user.name => "${user.name}@example.com"
  }
}

output "active_users" {
  value = local.active_users
}

output "user_emails" {
  value = local.user_emails
}
```

### Example 5: Dynamic Security Group Rules with Complex Logic

```hcl
locals {
  services = {
    "web" = {
      description = "Web Service Security Group


Understood! Let's create a series of complex examples that focus solely on local values and outputs, showcasing advanced data manipulation and transformation using Terraform's built-in functions.

### Example 1: Advanced Data Transformation and Validation

```hcl
locals {
  raw_data = {
    "server1" = {
      ip     = "10.0.1.10"
      memory = "16GB"
      cpu    = "4 cores"
    }
    "server2" = {
      ip     = "10.0.1.11"
      memory = "32GB"
      cpu    = "8 cores"
    }
  }

  transformed_data = {
    for k, v in local.raw_data : k => {
      ip     = v.ip
      memory = tonumber(replace(v.memory, "/GB/", ""))
      cpu    = tonumber(replace(v.cpu, "/ cores/", ""))
    }
  }

  validated_data = {
    for k, v in local.transformed_data : k => {
      ip     = v.ip
      memory = v.memory > 10 ? v.memory : null
      cpu    = v.cpu > 2 ? v.cpu : null
    }
  }
}

output "validated_data" {
  value = local.validated_data
}
```

### Example 2: Nested Data Aggregation and Grouping

```hcl
locals {
  transactions = [
    { user = "Alice", amount = 100, category = "food" },
    { user = "Bob", amount = 200, category = "travel" },
    { user = "Alice", amount = 150, category = "travel" },
    { user = "Charlie", amount = 300, category = "food" },
  ]

  user_transactions = {
    for user in distinct([for t in local.transactions : t.user]) : user => [
      for t in local.transactions : t if t.user == user
    ]
  }

  user_totals = {
    for user, txs in local.user_transactions : user => sum([for t in txs : t.amount])
  }
}

output "user_totals" {
  value = local.user_totals
}
```

### Example 3: Complex Data Filtering and Mapping with Conditional Logic

```hcl
locals {
  logs = [
    { timestamp = "2023-01-01T00:00:00Z", message = "Starting service", level = "info" },
    { timestamp = "2023-01-01T00:01:00Z", message = "Service started", level = "info" },
    { timestamp = "2023-01-01T00:02:00Z", message = "Error: connection lost", level = "error" },
    { timestamp = "2023-01-01T00:03:00Z", message = "Warning: high latency", level = "warning" },
  ]

  filtered_logs = [
    for log in local.logs : log if log.level == "error" || log.level == "warning"
  ]

  formatted_logs = [
    for log in local.filtered_logs : "${log.timestamp} - [${log.level}] ${log.message}"
  ]
}

output "formatted_logs" {
  value = join("\n", local.formatted_logs)
}
```

### Example 4: Advanced Data Normalization and Deduplication

```hcl
locals {
  raw_data = [
    { id = 1, name = "Alice", email = "alice@example.com" },
    { id = 2, name = "Bob", email = "bob@example.com" },
    { id = 1, name = "Alice", email = "alice@example.com" },
    { id = 3, name = "Charlie", email = "charlie@example.com" },
  ]

  unique_data = {
    for item in local.raw_data : "${item.id}-${item.email}" => item...
  }

### Example 5: Complex Data Sharding and Partitioning

```hcl
locals {
  large_dataset = [
    { id = 1, name = "Alice", data = "Lorem ipsum dolor sit amet" },
    { id = 2, name = "Bob", data = "Consectetur adipiscing elit" },
    { id = 3, name = "Charlie", data = "Sed do eiusmod tempor incididunt" },
    { id = 4, name = "David", data = "Ut labore et dolore magna aliqua" },
    { id = 5, name = "Eve", data = "Ut enim ad minim veniam" },
    { id = 6, name = "Frank", data = "Quis nostrud exercitation ullamco" },
    { id = 7, name = "Grace", data = "Laboris nisi ut aliquip ex ea commodo consequat" },
    { id = 8, name = "Hank", data = "Duis aute irure dolor in reprehenderit" },
    { id = 9, name = "Ivy", data = "In voluptate velit esse cillum dolore" },
    { id = 10, name = "Jack", data = "Eu fugiat nulla pariatur" },
  ]

  shard_size = 3

  sharded_data = [
    for i in range(0, length(local.large_dataset), local.shard_size) :
    slice(local.large_dataset, i, i + local.shard_size)
  ]

  partitioned_data = {
    for idx, shard in zip(range(length(local.sharded_data)), local.sharded_data) : "partition_${idx}" => shard
  }
}

output "partitioned_data" {
  value = local.partitioned_data
}
```

### Explanation:

1. **Example 1: Advanced Data Transformation and Validation**
   - Transforms raw data by converting memory and CPU strings to numbers.
   - Validates the transformed data to ensure memory and CPU values meet minimum thresholds.

2. **Example 2: Nested Data Aggregation and Grouping**
   - Aggregates transactions by user and calculates total amounts.
   - Demonstrates nested loops and aggregation functions.

3. **Example 3: Complex Data Filtering and Mapping with Conditional Logic**
   - Filters logs based on severity levels (error and warning).
   - Formats the filtered logs into a readable string.

4. **Example 4: Advanced Data Normalization and Deduplication**
   - Normalizes raw data by deduplicating based on a composite key (id and email).
   - Uses a map to ensure uniqueness.

5. **Example 5: Complex Data Sharding and Partitioning**
   - Shards a large dataset into smaller subsets of a specified size.
   - Partitions the sharded data into named partitions.
   - Demonstrates advanced data manipulation techniques.

These examples showcase the power of Terraform's local values and output transformations, allowing for complex data manipulation and transformation without relying on cloud provider resources. Each example builds on the previous one, demonstrating increasingly sophisticated techniques.



Absolutely! Let's push the boundaries even further with a more complex example that combines nested loops, advanced Terraform functions, and intricate data transformations. This example will demonstrate how to process a multi-dimensional dataset, apply complex transformations, and generate detailed outputs.

### Example: Multi-Dimensional Data Processing and Transformation

```hcl
locals {
  raw_data = {
    "region1" = {
      "cluster1" = {
        nodes = [
          { id = 1, type = "small", status = "active" },
          { id = 2, type = "medium", status = "inactive" },
        ]
      }
      "cluster2" = {
        nodes = [
          { id = 3, type = "large", status = "active" },
          { id = 4, type = "small", status = "active" },
        ]
      }
    }
    "region2" = {
      "cluster3" = {
        nodes = [
          { id = 5, type = "medium", status = "active" },
          { id = 6, type = "large", status = "inactive" },
        ]
      }
      "cluster4" = {
        nodes = [
          { id = 7, type = "small", status = "active" },
          { id = 8, type = "medium", status = "active" },
        ]
      }
    }
  }

  active_nodes = flatten([
    for region, clusters in local.raw_data : [
      for cluster, details in clusters : [
        for node in details.nodes : node if node.status == "active"
      ]
    ]
  ])

  node_counts = {
    for node in local.active_nodes : node.type => coalesce(lookup(local.node_counts, node.type, 0), 0) + 1...
  }

  node_details = {
    for node in local.active_nodes : node.id => {
      type   = node.type
      status = node.status
    }
  }

  region_summary = {
    for region, clusters in local.raw_data : region => {
      active_nodes = length([
        for cluster, details in clusters : [
          for node in details.nodes : node if node.status == "active"
        ]
      ])
      total_nodes = length(flatten([
        for cluster, details in clusters : details.nodes
      ]))
    }
  }
}

output "active_nodes" {
  value = local.active_nodes
}

output "node_counts" {
  value = local.node_counts
}

output "node_details" {
  value = local.node_details
}

output "region_summary" {
  value = local.region_summary
}
```

### Explanation:

1. **Raw Data Structure**:
   - A multi-dimensional map representing regions, clusters, and nodes.
   - Each node has an ID, type, and status.

2. **Active Nodes Extraction**:
   - Uses nested loops to flatten the structure and extract only active nodes.
   - The `flatten` function is used to merge lists from different levels.

3. **Node Counts by Type**:
   - Counts the number of active nodes by their type.
   - Uses the `coalesce` function to handle missing keys and increment counts dynamically.

4. **Node Details**:
   - Creates a map of node details keyed by node ID.
   - Filters out inactive nodes.

5. **Region Summary**:
   - Generates a summary of active and total nodes per region.
   - Uses nested loops and the `length` function to count nodes.

6. **Outputs**:
   - `active_nodes`: Lists all active nodes.
   - `node_counts`: Counts of active nodes by type.
   - `node_details`: Detailed information about each active node.
   - `region_summary`: Summary of node counts per region.

This example demonstrates the power of Terraform's local values and functions to process and transform complex, multi-dimensional data structures. It showcases advanced techniques such as nested loops, dynamic counting, and detailed data aggregation, providing a comprehensive view of the data.



Certainly! Let's create an even more advanced example that combines deeply nested loops, complex data transformations, and sophisticated Terraform functions. This example will process a hierarchical dataset, apply multiple layers of transformations, and generate detailed outputs.

### Example: Hierarchical Data Processing and Transformation

```hcl
locals {
  raw_data = {
    "region1" = {
      "cluster1" = {
        nodes = [
          { id = 1, type = "small", status = "active", metrics = { cpu = 20, memory = 16 } },
          { id = 2, type = "medium", status = "inactive", metrics = { cpu = 40, memory = 32 } },
        ]
      }
      "cluster2" = {
        nodes = [
          { id = 3, type = "large", status = "active", metrics = { cpu = 80, memory = 64 } },
          { id = 4, type = "small", status = "active", metrics = { cpu = 20, memory = 16 } },
        ]
      }
    }
    "region2" = {
      "cluster3" = {
        nodes = [
          { id = 5, type = "medium", status = "active", metrics = { cpu = 40, memory = 32 } },
          { id = 6, type = "large", status = "inactive", metrics = { cpu = 80, memory = 64 } },
        ]
      }
      "cluster4" = {
        nodes = [
          { id = 7, type = "small", status = "active", metrics = { cpu = 20, memory = 16 } },
          { id = 8, type = "medium", status = "active", metrics = { cpu = 40, memory = 32 } },
        ]
      }
    }
  }

  active_nodes = flatten([
    for region, clusters in local.raw_data : [
      for cluster, details in clusters : [
        for node in details.nodes : node if node.status == "active"
      ]
    ]
  ])

  node_metrics = {
    for node in local.active_nodes : node.id => {
      cpu    = node.metrics.cpu
      memory = node.metrics.memory
    }
  }

  region_metrics = {
    for region, clusters in local.raw_data : region => {
      cpu_total    = sum([
        for cluster, details in clusters : sum([
          for node in details.nodes : node.metrics.cpu if node.status == "active"
        ])
      ])
      memory_total = sum([
        for cluster, details in clusters : sum([
          for node in details.nodes : node.metrics.memory if node.status == "active"
        ])
      ])
    }
  }

  cluster_metrics = {
    for region, clusters in local.raw_data : region => {
      for cluster, details in clusters : cluster => {
        cpu_total    = sum([for node in details.nodes : node.metrics.cpu if node.status == "active"])
        memory_total = sum([for node in details.nodes : node.metrics.memory if node.status == "active"])
      }
    }
  }

  normalized_metrics = {
    for region, metrics in local.region_metrics : region => {
      cpu_normalized    = metrics.cpu_total / length(local.active_nodes)
      memory_normalized = metrics.memory_total / length(local.active_nodes)
    }
  }
}

output "active_nodes" {
  value = local.active_nodes
}

output "node_metrics" {
  value = local.node_metrics
}

output "region_metrics" {
  value = local.region_metrics
}

output "cluster_metrics" {
  value = local.cluster_metrics
}

output "normalized_metrics" {
  value = local.normalized_metrics
}
```

### Explanation:

1. **Raw Data Structure**:
   - A hierarchical map representing regions, clusters, and nodes.
   - Each node has an ID, type, status, and metrics (CPU and memory).

2. **Active Nodes Extraction**:
   - Uses nested loops to flatten the
  


Understood! Let's create a unique and innovative example that focuses on generating a dynamic configuration for a fictional monitoring system. This example will use Terraform's local values and functions to process a dataset of monitored services, apply complex rules, and generate a detailed monitoring configuration.

### Example: Dynamic Monitoring Configuration Generation

```hcl
locals {
  monitored_services = {
    "web-service" = {
      endpoints = [
        { path = "/health", method = "GET", expected_status = 200 },
        { path = "/api/v1/data", method = "POST", expected_status = 201 },
      ]
      alerts = [
        { type = "latency", threshold = 500, severity = "warning" },
        { type = "error_rate", threshold = 1, severity = "critical" },
      ]
    }
    "db-service" = {
      endpoints = [
        { path = "/metrics", method = "GET", expected_status = 200 },
      ]
      alerts = [
        { type = "latency", threshold = 1000, severity = "warning" },
        { type = "error_rate", threshold = 0.5, severity = "critical" },
      ]
    }
  }

  monitoring_config = {
    for service, details in local.monitored_services : service => {
      checks = [
        for endpoint in details.endpoints : {
          name = "${service}-${endpoint.path}-${endpoint.method}"
          type = "http"
          config = {
            url            = "http://${service}${endpoint.path}"
            method         = endpoint.method
            expected_status = endpoint.expected_status
          }
        }
      ]
      alerts = [
        for alert in details.alerts : {
          name = "${service}-${alert.type}-alert"
          type = alert.type
          config = {
            threshold = alert.threshold
            severity  = alert.severity
          }
        }
      ]
    }
  }

  consolidated_config = {
    checks = flatten([
      for service, config in local.monitoring_config : config.checks
    ])
    alerts = flatten([
      for service, config in local.monitoring_config : config.alerts
    ])
  }
}

output "monitoring_config" {
  value = local.monitoring_config
}

output "consolidated_config" {
  value = local.consolidated_config
}
```

### Explanation:

1. **Monitored Services Dataset**:
   - A map of services to be monitored, each with a list of endpoints and alerts.
   - Endpoints define the paths, methods, and expected status codes.
   - Alerts define the type, threshold, and severity of alerts.

2. **Dynamic Monitoring Configuration Generation**:
   - Uses nested loops to generate a detailed monitoring configuration for each service.
   - For each endpoint, creates a check with a unique name and HTTP configuration.
   - For each alert, creates an alert configuration with a unique name and threshold settings.

3. **Consolidated Configuration**:
   - Flattens the checks and alerts into a single list for easier consumption by the monitoring system.
   - Demonstrates how to consolidate multiple configurations into a single, unified structure.

4. **Outputs**:
   - `monitoring_config`: Detailed monitoring configuration for each service.
   - `consolidated_config`: Consolidated list of checks and alerts for all services.

This example showcases a unique use case of generating dynamic monitoring configurations using Terraform's local values and functions. It demonstrates advanced data manipulation, nested loops, and complex data transformation, providing a comprehensive and flexible monitoring setup.


Certainly! Let's create another unique example that focuses on generating a dynamic access control list (ACL) for a fictional network security system. This example will use Terraform's local values and functions to process a dataset of network rules, apply complex transformations, and generate a detailed ACL configuration.

### Example: Dynamic Network ACL Generation

```hcl
locals {
  network_rules = {
    "web-tier" = {
      inbound = [
        { protocol = "TCP", port = 80, source = "0.0.0.0/0", action = "allow" },
        { protocol = "TCP", port = 443, source = "0.0.0.0/0", action = "allow" },
      ]
      outbound = [
        { protocol = "TCP", port = 53, source = "10.0.0.0/8", action = "allow" },
        { protocol = "UDP", port = 53, source = "10.0.0.0/8", action = "allow" },
      ]
    }
    "db-tier" = {
      inbound = [
        { protocol = "TCP", port = 3306, source = "10.0.0.0/8", action = "allow" },
      ]
      outbound = [
        { protocol = "TCP", port = 80, source = "0.0.0.0/0", action = "deny" },
        { protocol = "TCP", port = 443, source = "0.0.0.0/0", action = "deny" },
      ]
    }
  }

  acl_config = {
    for tier, rules in local.network_rules : tier => {
      inbound = [
        for rule in rules.inbound : {
          name   = "${tier}-inbound-${rule.protocol}-${rule.port}"
          protocol = rule.protocol
          port     = rule.port
          source   = rule.source
          action   = rule.action
        }
      ]
      outbound = [
        for rule in rules.outbound : {
          name   = "${tier}-outbound-${rule.protocol}-${rule.port}"
          protocol = rule.protocol
          port     = rule.port
          source   = rule.source
          action   = rule.action
        }
      ]
    }
  }

  consolidated_acl = {
    inbound = flatten([
      for tier, config in local.acl_config : config.inbound
    ])
    outbound = flatten([
      for tier, config in local.acl_config : config.outbound
    ])
  }

  filtered_acl = {
    inbound = [
      for rule in local.consolidated_acl.inbound : rule if rule.action == "allow"
    ]
    outbound = [
      for rule in local.consolidated_acl.outbound : rule if rule.action == "allow"
    ]
  }

  sorted_acl = {
    inbound = sort(local.filtered_acl.inbound, "port")
    outbound = sort(local.filtered_acl.outbound, "port")
  }
}

output "acl_config" {
  value = local.acl_config
}

output "consolidated_acl" {
  value = local.consolidated_acl
}

output "filtered_acl" {
  value = local.filtered_acl
}

output "sorted_acl" {
  value = local.sorted_acl
}
```

### Explanation:

1. **Network Rules Dataset**:
   - A map of network tiers with inbound and outbound rules.
   - Each rule defines the protocol, port, source, and action (allow or deny).

2. **Dynamic ACL Configuration Generation**:
   - Uses nested loops to generate a detailed ACL configuration for each network tier.
   - For each rule, creates a unique name and applies the rule configuration.

3. **Consolidated ACL**:
   - Flattens the inbound and outbound rules into single lists for easier management.
   - Demonstrates how to consolidate multiple configurations into a single structure.

4. **Filtered ACL**:
   - Filters the consolidated ACL to include only rules with the action "allow".
   - Shows how to apply conditional logic to filter data.

5. **Sorted ACL**:
   - Sorts the filtered ACL by
