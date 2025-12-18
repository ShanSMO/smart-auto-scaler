# DQN Kubernetes Resource Scaler API Documentation

## Overview

The DQN Kubernetes Resource Scaler is a Flask-based REST API service that uses Deep Q-Network (DQN) machine learning to automatically scale Kubernetes pod resources based on real-time metrics. The API provides endpoints for configuration management, pod scaling operations, monitoring, and auto-scaling control.

## Base Information

- **Service**: DQN Kubernetes Resource Scaler
- **Version**: 1.0.0
- **Model**: Deep Q-Network (DQN) with multi-head architecture for CPU and Memory decisions
- **Framework**: Flask 3.0.0 with CORS support
- **Port**: Configurable (typically 5000)

## Table of Contents

- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [Endpoints](#endpoints)
- [Data Models](#data-models)
- [Configuration](#configuration)
- [Examples](#examples)

## Authentication

**Note**: This API currently does not implement authentication mechanisms. Access control should be handled at the infrastructure level (network policies, service mesh, etc.) when deployed in production environments.

## Error Handling

### Standard Error Response Format

```json
{
  "success": false,
  "error": "Error message description"
}
```

### HTTP Status Codes

- `200` - Success
- `400` - Bad Request (invalid parameters)
- `404` - Not Found (endpoint or resource not found)
- `500` - Internal Server Error
- `503` - Service Unavailable (Kubernetes client not initialized)

### Error Handler Routes

- `404` - Returns standard error format for unknown endpoints
- `500` - Returns standard error format for internal server errors

## Endpoints

### Service Information

#### GET /

Returns basic service information and available endpoints.

**Response:**
```json
{
  "service": "DQN Kubernetes Resource Scaler",
  "version": "1.0.0",
  "model": "Deep Q-Network (DQN)",
  "endpoints": {
    "GET /health": "Health check",
    "GET /config": "Get current configuration",
    "POST /config": "Update configuration",
    "GET /pods": "List all monitored pods",
    "GET /pods/<namespace>/<pod_name>": "Get specific pod info",
    "POST /scale/pod/<namespace>/<pod_name>": "Scale specific pod",
    "POST /scale/all": "Process and scale all pods",
    "GET /decisions": "Get recent scaling decisions",
    "GET /statistics": "Get scaling statistics",
    "POST /autoscale/start": "Start automatic scaling",
    "POST /autoscale/stop": "Stop automatic scaling",
    "GET /autoscale/status": "Get auto-scaler status",
    "POST /api/namespaces/<namespace>/pods/<pod_name>/resize": "Resize pod resources"
  }
}
```

#### GET /health

Health check endpoint to verify service status.

**Response:**
```json
{
  "status": "healthy",
  "service": "dqn-scaler",
  "timestamp": "2024-12-18T10:30:00.000Z"
}
```

### Configuration Management

#### GET /config

Retrieve current service configuration.

**Response:**
```json
{
  "model_path": "final-models/dqn_model.pth",
  "state_dim": 8,
  "action_dim": 3,
  "history_window": 5,
  "min_cpu": 0.1,
  "max_cpu": 8.0,
  "min_memory": 20,
  "max_memory": 16384,
  "scale_factor": 0.2,
  "dry_run": true,
  "in_cluster": false,
  "namespaces": ["default"],
  "auto_scale_enabled": true,
  "auto_scale_interval": 30,
  "scaling_cooldown": 30,
  "excluded_deployments": ["redis", "dqn-scaler", "metrics-collector"],
  "excluded_labels": {"app": "redis", "component": "redis"}
}
```

#### POST /config

Update service configuration.

**Request Body:**
```json
{
  "dry_run": false,
  "auto_scale_enabled": true,
  "auto_scale_interval": 60,
  "scale_factor": 0.3,
  "scaling_cooldown": 45,
  "namespaces": ["default", "production"],
  "excluded_deployments": ["redis", "database"],
  "excluded_labels": {"app": "system"}
}
```

**Allowed Configuration Fields:**
- `dry_run` (boolean)
- `auto_scale_enabled` (boolean)
- `auto_scale_interval` (integer, seconds)
- `scale_factor` (float)
- `scaling_cooldown` (integer, minutes)
- `namespaces` (array of strings)
- `excluded_deployments` (array of strings)
- `excluded_labels` (object)

**Response:**
```json
{
  "success": true,
  "message": "Configuration updated",
  "config": { /* updated configuration object */ }
}
```

### Pod Management

#### GET /pods

List all monitored pods across configured namespaces.

**Response:**
```json
{
  "success": true,
  "count": 5,
  "pods": [
    {
      "name": "nginx-deployment-abc123",
      "namespace": "default",
      "status": "Running",
      "creation_timestamp": "2024-12-18T10:00:00.000Z",
      "labels": {"app": "nginx", "version": "1.0"},
      "node_name": "worker-node-1",
      "pod_ip": "10.244.1.5",
      "containers": [
        {
          "name": "nginx",
          "image": "nginx:1.21",
          "resources": {
            "requests": {"cpu": "100m", "memory": "128Mi"},
            "limits": {"cpu": "500m", "memory": "512Mi"}
          }
        }
      ],
      "owner": {
        "kind": "ReplicaSet",
        "name": "nginx-deployment-xyz789"
      }
    }
  ]
}
```

#### GET /pods/\<namespace>/\<pod_name>

Get detailed information about a specific pod.

**Path Parameters:**
- `namespace` - Kubernetes namespace
- `pod_name` - Pod name

**Response:**
```json
{
  "success": true,
  "pod": "default/nginx-deployment-abc123",
  "metrics": {
    "cpu_usage_cores": 0.15,
    "memory_usage_mb": 256.5
  },
  "resources": {
    "cpu_requests_cores": 0.1,
    "memory_requests_mb": 128
  }
}
```

### Scaling Operations

#### POST /scale/pod/\<namespace>/\<pod_name>

Process and potentially scale a specific pod using DQN model.

**Path Parameters:**
- `namespace` - Kubernetes namespace
- `pod_name` - Pod name

**Response:**
```json
{
  "success": true,
  "result": {
    "pod_name": "nginx-deployment-abc123",
    "namespace": "default",
    "timestamp": "2024-12-18T10:30:00.000Z",
    "current_metrics": {
      "cpu_usage": 0.15,
      "memory_usage_mb": 256.5,
      "cpu_limit": 0.5,
      "memory_limit_mb": 512.0
    },
    "predictions": {
      "cpu_action": "INCREASE",
      "memory_action": "MAINTAIN",
      "cpu_action_index": 2,
      "memory_action_index": 1
    },
    "confidence": {
      "cpu": 0.85,
      "memory": 0.92
    },
    "future_predictions": {
      "next_step_cpu": 0.18,
      "next_step_memory": 250.0
    },
    "q_values": {
      "cpu_q_values": [0.1, 0.3, 0.9],
      "memory_q_values": [0.2, 0.95, 0.4]
    },
    "resource_changes": {
      "cpu": {
        "current": 0.5,
        "new": 0.6,
        "change_percent": 20.0,
        "action": "INCREASE"
      },
      "memory": {
        "current": 512.0,
        "new": 512.0,
        "change_percent": 0.0,
        "action": "MAINTAIN"
      }
    },
    "can_scale": true,
    "explanation": {
      "reasoning": "High CPU utilization detected. Memory usage is stable.",
      "factors": ["cpu_trend_increasing", "memory_stable"]
    }
  }
}
```

#### POST /scale/all

Process and scale all monitored pods automatically.

**Response:**
```json
{
  "success": true,
  "processed": 3,
  "results": [
    {
      "pod_name": "nginx-deployment-abc123",
      "status": "success",
      "message": "Scaling executed successfully",
      "actions": {
        "cpu": "INCREASE",
        "memory": "MAINTAIN"
      }
    },
    {
      "pod_name": "redis-cache-xyz789",
      "status": "skipped",
      "reason": "cooldown_period",
      "message": "Pod redis-cache-xyz789 is in cooldown period"
    }
  ],
  "statistics": {
    "overview": {
      "total_decisions": 150,
      "total_pods_monitored": 8,
      "applied_scalings": 45,
      "scaling_rate": 30.0
    },
    "action_distribution": {
      "cpu_actions": {"DECREASE": 10, "MAINTAIN": 80, "INCREASE": 60},
      "memory_actions": {"DECREASE": 5, "MAINTAIN": 120, "INCREASE": 25}
    }
  },
  "timestamp": "2024-12-18T10:30:00.000Z"
}
```

### Pod Resizing

#### POST /api/namespaces/\<namespace>/pods/\<pod_name>/resize

Resize pod container resources in-place using Kubernetes resize feature.

**Path Parameters:**
- `namespace` - Kubernetes namespace
- `pod_name` - Pod name

**Query Parameters:**
- `enable_horizontal_fallback` (optional, default: true) - Enable fallback to deployment updates

**Request Body:**
```json
{
  "containers": {
    "nginx": {
      "requests": {
        "cpu": "200m",
        "memory": "256Mi"
      },
      "limits": {
        "cpu": "800m",
        "memory": "1Gi"
      }
    }
  }
}
```

**Response (Success):**
```json
{
  "message": "Successfully resized pod nginx-deployment-abc123 vertically",
  "pod": "nginx-deployment-abc123",
  "namespace": "default",
  "scaling_method": "vertical",
  "details": {
    "pod_name": "nginx-deployment-abc123",
    "namespace": "default",
    "container_resources": { /* resource specifications */ }
  },
  "timestamp": "2024-12-18T10:30:00.000Z"
}
```

**Response (Fallback):**
```json
{
  "message": "Successfully updated deployment nginx-deployment resources",
  "pod": "nginx-deployment-abc123",
  "namespace": "default",
  "scaling_method": "deployment_update",
  "details": {
    "pod_name": "nginx-deployment-abc123",
    "deployment_name": "nginx-deployment",
    "namespace": "default",
    "container_resources": { /* resource specifications */ },
    "reason": "Direct vertical scaling failed, updated via deployment"
  },
  "timestamp": "2024-12-18T10:30:00.000Z"
}
```

### Monitoring and Analytics

#### GET /decisions

Get recent scaling decisions across all pods.

**Query Parameters:**
- `limit` (optional, default: 50) - Maximum number of decisions to return

**Response:**
```json
{
  "success": true,
  "count": 25,
  "decisions": [
    {
      "pod_name": "nginx-deployment-abc123",
      "namespace": "default",
      "timestamp": "2024-12-18T10:30:00.000Z",
      "cpu_action": "INCREASE",
      "memory_action": "MAINTAIN",
      "confidence": {"cpu": 0.85, "memory": 0.92},
      "can_scale": true,
      "current_metrics": {
        "cpu_usage": 0.15,
        "memory_usage_mb": 256.5,
        "cpu_limit": 0.5,
        "memory_limit_mb": 512.0
      },
      "utilization": {
        "cpu_percentage": 30.0,
        "memory_percentage": 50.1
      },
      "future_predictions": {
        "next_step_cpu": 0.18,
        "next_step_memory": 250.0
      }
    }
  ]
}
```

#### GET /statistics

Get comprehensive scaling statistics and performance metrics.

**Response:**
```json
{
  "success": true,
  "statistics": {
    "overview": {
      "total_decisions": 500,
      "total_pods_monitored": 12,
      "applied_scalings": 120,
      "scaling_rate": 24.0
    },
    "action_distribution": {
      "cpu_actions": {
        "DECREASE": 50,
        "MAINTAIN": 350,
        "INCREASE": 100
      },
      "memory_actions": {
        "DECREASE": 25,
        "MAINTAIN": 400,
        "INCREASE": 75
      }
    },
    "resource_utilization": {
      "average_cpu_usage": 0.45,
      "average_memory_usage_mb": 512.75
    },
    "model_performance": {
      "average_cpu_confidence": 0.87,
      "average_memory_confidence": 0.91,
      "model_type": "Multi-Head DQN"
    },
    "system_status": {
      "dry_run_mode": false,
      "cooldown_period_minutes": 5,
      "scale_factor": 0.2,
      "kubernetes_available": true
    },
    "timestamp": "2024-12-18T10:30:00.000Z"
  }
}
```

### Auto-Scaling Control

#### POST /autoscale/start

Start the automatic scaling service.

**Response:**
```json
{
  "success": true,
  "message": "Auto-scaling started",
  "interval": 30
}
```

#### POST /autoscale/stop

Stop the automatic scaling service.

**Response:**
```json
{
  "success": true,
  "message": "Auto-scaling stopped"
}
```

#### GET /autoscale/status

Get current auto-scaling status.

**Response:**
```json
{
  "enabled": true,
  "running": true,
  "interval_seconds": 30,
  "thread_alive": true
}
```

### Model Information

#### GET /model/info

Get information about the DQN model and its configuration.

**Response:**
```json
{
  "model_type": "Deep Q-Network (DQN)",
  "model_path": "final-models/dqn_model.pth",
  "state_dim": 8,
  "action_dim": 3,
  "actions": {
    "0": "DECREASE",
    "1": "MAINTAIN",
    "2": "INCREASE"
  },
  "state_features": [
    "CPU usage (normalized)",
    "Memory usage (normalized)",
    "Network latency (placeholder)",
    "Container count (normalized)",
    "CPU trend",
    "Memory trend",
    "CPU allocation (normalized)",
    "Memory allocation (normalized)"
  ]
}
```

## Data Models

### Pod Metrics
```typescript
interface PodMetrics {
  pod_name: string;
  namespace: string;
  cpu_usage: number;           // Current CPU usage in cores
  memory_usage_mb: number;     // Current memory usage in MB
  cpu_limit: number;           // CPU limit in cores
  memory_limit_mb: number;     // Memory limit in MB
  phase: string;               // Pod phase (Running, Pending, etc.)
  node_name: string;           // Node where pod is scheduled
}
```

### Scaling Decision
```typescript
interface ScalingDecision {
  pod_name: string;
  namespace: string;
  timestamp: string;           // ISO 8601 format
  current_metrics: PodMetrics;
  predictions: {
    cpu_action: "DECREASE" | "MAINTAIN" | "INCREASE";
    memory_action: "DECREASE" | "MAINTAIN" | "INCREASE";
    cpu_action_index: 0 | 1 | 2;
    memory_action_index: 0 | 1 | 2;
  };
  confidence: {
    cpu: number;               // 0.0 to 1.0
    memory: number;            // 0.0 to 1.0
  };
  future_predictions: {
    next_step_cpu: number;
    next_step_memory: number;
  };
  q_values: {
    cpu_q_values: number[];    // Array of 3 Q-values
    memory_q_values: number[]; // Array of 3 Q-values
  };
  resource_changes: {
    cpu: ResourceChange;
    memory: ResourceChange;
  };
  can_scale: boolean;
  explanation: {
    reasoning: string;
    factors: string[];
  };
}
```

### Resource Change
```typescript
interface ResourceChange {
  current: number;
  new: number;
  change_percent: number;
  action: "DECREASE" | "MAINTAIN" | "INCREASE";
}
```

### Container Resources
```typescript
interface ContainerResources {
  [containerName: string]: {
    requests?: {
      cpu?: string;      // e.g., "100m", "0.5"
      memory?: string;   // e.g., "128Mi", "1Gi"
    };
    limits?: {
      cpu?: string;      // e.g., "500m", "2"
      memory?: string;   // e.g., "512Mi", "4Gi"
    };
  };
}
```

## Configuration

### Environment Variables

The service can be configured using environment variables:

- `MODEL_PATH` - Path to the DQN model file
- `DRY_RUN` - Enable dry run mode (true/false)
- `AUTO_SCALE_ENABLED` - Enable automatic scaling (true/false)
- `AUTO_SCALE_INTERVAL` - Scaling interval in seconds
- `NAMESPACES` - Comma-separated list of namespaces to monitor
- `EXCLUDED_DEPLOYMENTS` - Comma-separated list of deployments to exclude
- `KUBERNETES_IN_CLUSTER` - Run in Kubernetes cluster mode (true/false)

### Default Configuration

```json
{
  "model_path": "final-models/dqn_model.pth",
  "state_dim": 8,
  "action_dim": 3,
  "history_window": 5,
  "min_cpu": 0.1,
  "max_cpu": 8.0,
  "min_memory": 20,
  "max_memory": 16384,
  "scale_factor": 0.2,
  "dry_run": true,
  "in_cluster": false,
  "namespaces": ["default"],
  "auto_scale_enabled": true,
  "auto_scale_interval": 30,
  "scaling_cooldown": 30,
  "excluded_deployments": [
    "redis", 
    "dqn-scaler", 
    "metrics-collector", 
    "load-generator", 
    "dqn-scaler-dashboard"
  ],
  "excluded_labels": {
    "app": "redis", 
    "component": "redis"
  }
}
```

## Examples

### Basic Usage

#### 1. Check Service Health
```bash
curl http://localhost:5000/health
```

#### 2. Get Current Configuration
```bash
curl http://localhost:5000/config
```

#### 3. Enable Auto-scaling
```bash
curl -X POST http://localhost:5000/config \
  -H "Content-Type: application/json" \
  -d '{"auto_scale_enabled": true, "dry_run": false}'
```

#### 4. Scale a Specific Pod
```bash
curl -X POST http://localhost:5000/scale/pod/default/nginx-deployment-abc123
```

#### 5. Resize Pod Resources
```bash
curl -X POST http://localhost:5000/api/namespaces/default/pods/nginx-deployment-abc123/resize \
  -H "Content-Type: application/json" \
  -d '{
    "containers": {
      "nginx": {
        "requests": {
          "cpu": "200m",
          "memory": "256Mi"
        },
        "limits": {
          "cpu": "800m",
          "memory": "1Gi"
        }
      }
    }
  }'
```

#### 6. Get Scaling Statistics
```bash
curl http://localhost:5000/statistics
```

#### 7. Get Recent Decisions
```bash
curl http://localhost:5000/decisions?limit=10
```

### Advanced Usage

#### Update Multiple Configuration Values
```bash
curl -X POST http://localhost:5000/config \
  -H "Content-Type: application/json" \
  -d '{
    "auto_scale_enabled": true,
    "auto_scale_interval": 60,
    "namespaces": ["default", "production"],
    "excluded_deployments": ["redis", "database", "monitoring"],
    "scaling_cooldown": 45
  }'
```

#### Monitor Auto-scaling Status
```bash
# Start auto-scaling
curl -X POST http://localhost:5000/autoscale/start

# Check status
curl http://localhost:5000/autoscale/status

# Stop auto-scaling
curl -X POST http://localhost:5000/autoscale/stop
```

## Dependencies

### Core Dependencies
- Flask 3.0.0 - Web framework
- Flask-CORS 4.0.0 - Cross-origin resource sharing
- Kubernetes 28.1.0 - Kubernetes client library
- NumPy ≥1.24.0 - Numerical computations
- PyTorch (CPU) - Deep learning framework
- Gunicorn 21.2.0 - WSGI server
- Requests ≥2.31.0 - HTTP client

### Kubernetes Requirements
- Kubernetes 1.20+ (recommended 1.27+ for in-place pod resize)
- Metrics Server installed for resource metrics
- RBAC permissions for pod and deployment management

## Deployment

### Local Development
```bash
pip install -r requirements.txt
python app.py
```

### Docker Deployment
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
```

### Kubernetes Deployment
Requires appropriate RBAC permissions for:
- Reading pods, deployments, replicasets
- Updating pod resources
- Accessing metrics server APIs

## Troubleshooting

### Common Issues

1. **Kubernetes Client Not Available**
   - Ensure kubeconfig is properly configured
   - Check RBAC permissions
   - Verify metrics server is running

2. **DQN Model Loading Errors**
   - Verify model file exists at specified path
   - Check PyTorch version compatibility
   - Ensure model dimensions match configuration

3. **Pod Resize Failures**
   - Kubernetes 1.27+ required for in-place resize
   - Check pod has resource specifications
   - Verify pod is in Running state

4. **Auto-scaling Not Working**
   - Check `auto_scale_enabled` is true
   - Verify thread is alive via `/autoscale/status`
   - Review logs for error messages

### Monitoring and Logging

The service uses Python's logging module with configurable levels. Key log sources:
- Scaling decisions and actions
- Kubernetes API interactions
- DQN model predictions
- Configuration changes
- Error conditions

For production deployments, consider:
- Centralized logging (ELK stack, Fluentd)
- Metrics collection (Prometheus)
- Alerting on scaling failures
- Resource usage monitoring
