# Week 4: AI Platforms & Infrastructure - Complete Guide

**📅 Week:** 4 of 5  
**⏱️ Estimated Time:** 10-12 hours  
**🎯 Difficulty:** Intermediate to Advanced  
**📝 Type:** Infrastructure & Platform Engineering

---

## Table of Contents

1. [Introduction](#introduction)
2. [AI Platform Architecture](#ai-platform-architecture)
3. [Inference Gateways](#inference-gateways)
4. [Cost Management & Optimization](#cost-management--optimization)
5. [Centralized vs. Federated Architecture](#centralized-vs-federated-architecture)
6. [Observability & Monitoring](#observability--monitoring)
7. [Scalability & Performance](#scalability--performance)
8. [Security & Compliance](#security--compliance)
9. [Hands-On Exercises](#hands-on-exercises)
10. [Practice Question Bank](#practice-question-bank)
11. [Self-Assessment Checklist](#self-assessment-checklist)
12. [Summary & Key Takeaways](#summary--key-takeaways)
13. [Further Reading](#further-reading)

---

## Introduction

Welcome to Week 4 of the InfoQ Certified AI Engineering Program. This week focuses on **AI Platforms & Infrastructure** - the foundation that makes AI systems production-ready, scalable, and cost-effective.

### Learning Objectives

By the end of this week, you will be able to:

✅ **Design** AI platform architectures for enterprise scale  
✅ **Implement** inference gateways for model serving  
✅ **Optimize** GPU costs and token usage  
✅ **Decide** between centralized and federated architectures  
✅ **Build** comprehensive observability for AI systems  
✅ **Scale** AI infrastructure horizontally and vertically  
✅ **Ensure** security and compliance for AI platforms  
✅ **Plan** workload routing for latency-critical tasks  

### Why AI Platforms Matter

> 💡 **The Platform Imperative:** Building AI models is only half the battle. Production AI requires robust platforms that handle inference, scaling, cost management, and observability at scale.

**Key Statistics:**
- **AI infrastructure spending** reached $62B in 2024 (Gartner)
- **Inference costs** can be 10-100x higher than training costs over time
- **Poor platform design** leads to 3-5x higher operational costs
- **Observability gaps** cause 60% of production AI incidents

### The AI Platform Landscape

```
┌──────────────────────────────────────────────────────────┐
│              AI Platform Architecture                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Application Layer                                       │
│  ┌─────────────────────────────────────────┐             │
│  │  AI Applications & Agents               │             │
│  └─────────────────────────────────────────┘             │
│                                                          │
│  Platform Layer                                          │
│  ┌──────────────┬──────────────┬──────────────┐         │
│  │  Inference   │   Model      │   Feature    │         │
│  │  Gateway     │   Registry   │   Store      │         │
│  └──────────────┴──────────────┴──────────────┘         │
│                                                          │
│  Infrastructure Layer                                    │
│  ┌──────────────┬──────────────┬──────────────┐         │
│  │   Compute    │   Storage    │   Network    │         │
│  │  (GPUs/TPUs) │  (Vector DB) │  (API GW)    │         │
│  └──────────────┴──────────────┴──────────────┘         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## AI Platform Architecture

### Core Components

#### 1. Inference Gateway

The entry point for all AI model requests.

```python
class InferenceGateway:
    """
    Central gateway for AI model inference
    """
    def __init__(self, config):
        self.models = {}
        self.load_balancer = LoadBalancer(config['load_balancing'])
        self.cache = Cache(config['cache'])
        self.rate_limiter = RateLimiter(config['rate_limits'])
        self.circuit_breaker = CircuitBreaker(config['circuit_breaker'])
    
    def register_model(self, model_name, model_config):
        """Register model with gateway"""
        self.models[model_name] = {
            'config': model_config,
            'endpoints': self.create_model_endpoints(model_config),
            'health_status': 'healthy',
            'metrics': MetricsCollector()
        }
    
    def route_request(self, model_name, request):
        """
        Route inference request to appropriate model
        """
        # Check cache
        cache_key = self.generate_cache_key(model_name, request)
        cached_response = self.cache.get(cache_key)
        if cached_response:
            return cached_response
        
        # Rate limiting
        rate_check = self.rate_limiter.check(request.client_id)
        if not rate_check['allowed']:
            return {
                'error': 'Rate limit exceeded',
                'retry_after': rate_check['retry_after']
            }
        
        # Get model
        model = self.models.get(model_name)
        if not model:
            return {'error': 'Model not found'}
        
        # Circuit breaker check
        if not self.circuit_breaker.can_execute(model_name):
            return {'error': 'Model temporarily unavailable'}
        
        # Load balance
        endpoint = self.load_balancer.select_endpoint(model['endpoints'])
        
        try:
            # Execute inference
            response = self.execute_inference(endpoint, request)
            
            # Record success
            self.circuit_breaker.record_success(model_name)
            model['metrics'].record_success(request, response)
            
            # Cache response
            self.cache.set(cache_key, response, ttl=300)
            
            return response
        
        except Exception as e:
            # Record failure
            self.circuit_breaker.record_failure(model_name)
            model['metrics'].record_failure(request, e)
            
            return {'error': str(e)}
    
    def execute_inference(self, endpoint, request):
        """Execute inference on model endpoint"""
        # Implementation depends on model type
        pass
```

#### 2. Model Registry

Central repository for model versions and metadata.

```python
class ModelRegistry:
    """
    Central model registry
    """
    def __init__(self, storage_backend):
        self.storage = storage_backend
        self.models = {}
    
    def register_model(self, model_name, version, model_path, metadata):
        """Register new model version"""
        model_id = f"{model_name}:{version}"
        
        model_entry = {
            'id': model_id,
            'name': model_name,
            'version': version,
            'path': model_path,
            'metadata': metadata,
            'created_at': datetime.now().isoformat(),
            'status': 'registered',
            'metrics': {}
        }
        
        # Store model
        self.storage.store_model(model_path, model_id)
        
        # Register in catalog
        self.models[model_id] = model_entry
        
        return model_id
    
    def get_model(self, model_name, version=None):
        """Get model by name and version"""
        if version:
            model_id = f"{model_name}:{version}"
            return self.models.get(model_id)
        else:
            # Get latest version
            matching = [m for m in self.models.values() if m['name'] == model_name]
            if not matching:
                return None
            
            # Sort by version
            matching.sort(key=lambda x: x['version'], reverse=True)
            return matching[0]
    
    def list_models(self):
        """List all registered models"""
        return list(self.models.values())
    
    def deprecate_model(self, model_name, version):
        """Deprecate model version"""
        model_id = f"{model_name}:{version}"
        if model_id in self.models:
            self.models[model_id]['status'] = 'deprecated'
            return True
        return False
```

#### 3. Feature Store

Shared feature repository for ML models.

```python
class FeatureStore:
    """
    Feature store for ML models
    """
    def __init__(self, online_store, offline_store):
        self.online_store = online_store  # Redis, Cassandra
        self.offline_store = offline_store  # S3, BigQuery
        self.features = {}
    
    def register_feature(self, feature_name, feature_def):
        """Register feature definition"""
        self.features[feature_name] = {
            'name': feature_name,
            'definition': feature_def,
            'created_at': datetime.now().isoformat()
        }
    
    def get_feature(self, feature_name, entity_id):
        """Get feature value for entity"""
        # Try online store first (low latency)
        value = self.online_store.get(feature_name, entity_id)
        
        if value is None:
            # Fall back to offline store
            value = self.offline_store.get(feature_name, entity_id)
            
            # Update online store
            self.online_store.set(feature_name, entity_id, value)
        
        return value
    
    def get_features(self, feature_names, entity_id):
        """Get multiple features for entity"""
        return {
            name: self.get_feature(name, entity_id)
            for name in feature_names
        }
    
    def compute_feature(self, feature_name, entity_id):
        """Compute feature value"""
        feature_def = self.features[feature_name]
        
        # Execute feature computation
        value = self.execute_computation(feature_def, entity_id)
        
        # Store in both stores
        self.online_store.set(feature_name, entity_id, value)
        self.offline_store.set(feature_name, entity_id, value)
        
        return value
```

### Platform Architecture Patterns

#### Pattern 1: Monolithic Platform

All components in a single platform.

```python
class MonolithicAIPlatform:
    """
    All-in-one AI platform
    """
    def __init__(self):
        self.inference_gateway = InferenceGateway()
        self.model_registry = ModelRegistry()
        self.feature_store = FeatureStore()
        self.monitoring = MonitoringSystem()
    
    def deploy_model(self, model_config):
        """Deploy model to platform"""
        # Register model
        model_id = self.model_registry.register_model(**model_config)
        
        # Deploy to inference gateway
        self.inference_gateway.register_model(model_id, model_config)
        
        # Setup monitoring
        self.monitoring.setup_model_monitoring(model_id)
        
        return model_id
```

**Pros:**
- Simple to manage
- Unified monitoring
- Easier debugging

**Cons:**
- Single point of failure
- Harder to scale components independently
- Tight coupling

#### Pattern 2: Microservices Platform

Separate services for each component.

```python
class MicroservicesAIPlatform:
    """
    Microservices-based AI platform
    """
    def __init__(self):
        self.inference_service = InferenceService()
        self.registry_service = RegistryService()
        self.feature_service = FeatureService()
        self.api_gateway = APIGateway()
    
    def deploy_model(self, model_config):
        """Deploy model via microservices"""
        # Register via registry service
        model_id = self.registry_service.register(model_config)
        
        # Deploy via inference service
        self.inference_service.deploy(model_id, model_config)
        
        # Setup feature service
        self.feature_service.setup_model_features(model_id)
        
        return model_id
```

**Pros:**
- Independent scaling
- Technology flexibility
- Fault isolation

**Cons:**
- More complex
- Network overhead
- Distributed debugging

---

## Inference Gateways

### Gateway Patterns

#### Pattern 1: Simple Proxy Gateway

```python
class SimpleProxyGateway:
    """Simple proxy to model servers"""
    def __init__(self, model_endpoints):
        self.endpoints = model_endpoints
    
    def route(self, model_name, request):
        """Route to model endpoint"""
        endpoint = self.endpoints.get(model_name)
        if not endpoint:
            return {'error': 'Model not found'}
        
        response = requests.post(endpoint, json=request)
        return response.json()
```

#### Pattern 2: Load-Balanced Gateway

```python
class LoadBalancedGateway:
    """Gateway with load balancing"""
    def __init__(self):
        self.models = {}
        self.load_balancers = {}
    
    def register_model(self, model_name, endpoints, strategy='round_robin'):
        """Register model with multiple endpoints"""
        self.models[model_name] = {
            'endpoints': endpoints,
            'strategy': strategy
        }
        
        self.load_balancers[model_name] = LoadBalancer(
            endpoints,
            strategy=strategy
        )
    
    def route(self, model_name, request):
        """Route with load balancing"""
        model = self.models.get(model_name)
        if not model:
            return {'error': 'Model not found'}
        
        # Select endpoint
        endpoint = self.load_balancers[model_name].select()
        
        # Execute
        response = requests.post(endpoint, json=request)
        return response.json()
```

#### Pattern 3: Intelligent Gateway

```python
class IntelligentGateway:
    """Gateway with intelligent routing"""
    def __init__(self):
        self.models = {}
        self.routing_rules = {}
        self.cache = Cache()
    
    def register_model(self, model_name, config):
        """Register model with routing rules"""
        self.models[model_name] = config
        
        # Setup routing rules
        self.routing_rules[model_name] = {
            'latency_threshold': config.get('latency_threshold', 100),
            'cost_threshold': config.get('cost_threshold', 0.01),
            'fallback_model': config.get('fallback_model')
        }
    
    def route(self, model_name, request):
        """Intelligent routing based on multiple factors"""
        rules = self.routing_rules.get(model_name)
        
        # Check cache
        cache_key = self.generate_cache_key(model_name, request)
        cached = self.cache.get(cache_key)
        if cached:
            return cached
        
        # Select model version based on request
        model_version = self.select_model_version(model_name, request)
        
        # Execute with monitoring
        start_time = time.time()
        try:
            response = self.execute_inference(model_version, request)
            latency = (time.time() - start_time) * 1000
            
            # Check if latency is acceptable
            if latency > rules['latency_threshold']:
                # Try fallback
                if rules['fallback_model']:
                    response = self.execute_inference(rules['fallback_model'], request)
            
            # Cache response
            self.cache.set(cache_key, response, ttl=300)
            
            return response
        
        except Exception as e:
            # Fallback
            if rules['fallback_model']:
                return self.execute_inference(rules['fallback_model'], request)
            raise
    
    def select_model_version(self, model_name, request):
        """Select model version based on request"""
        # Implementation: choose between model versions
        # based on request complexity, user tier, etc.
        pass
```

### Gateway Policies

```python
class GatewayPolicies:
    """Define and enforce gateway policies"""
    def __init__(self):
        self.policies = {
            'rate_limits': {},
            'cost_caps': {},
            'pii_handling': {},
            'authentication': {}
        }
    
    def setup_rate_limiting(self, model_name, limits):
        """Setup rate limiting for model"""
        self.policies['rate_limits'][model_name] = {
            'requests_per_minute': limits.get('rpm', 100),
            'tokens_per_minute': limits.get('tpm', 100000),
            'concurrent_requests': limits.get('concurrent', 10)
        }
    
    def setup_cost_caps(self, model_name, caps):
        """Setup cost caps for model"""
        self.policies['cost_caps'][model_name] = {
            'max_cost_per_request': caps.get('per_request', 0.01),
            'max_cost_per_day': caps.get('per_day', 100.0),
            'max_cost_per_month': caps.get('per_month', 1000.0)
        }
    
    def setup_pii_handling(self, model_name, rules):
        """Setup PII handling rules"""
        self.policies['pii_handling'][model_name] = {
            'detect_pii': rules.get('detect', True),
            'redact_pii': rules.get('redact', False),
            'block_pii': rules.get('block', False)
        }
    
    def enforce_policies(self, model_name, request):
        """Enforce all policies for request"""
        violations = []
        
        # Check rate limits
        rate_check = self.check_rate_limit(model_name, request)
        if not rate_check['allowed']:
            violations.append(rate_check)
        
        # Check cost caps
        cost_check = self.check_cost_cap(model_name, request)
        if not cost_check['allowed']:
            violations.append(cost_check)
        
        # Check PII
        pii_check = self.check_pii(model_name, request)
        if not pii_check['allowed']:
            violations.append(pii_check)
        
        return {
            'allowed': len(violations) == 0,
            'violations': violations
        }
```

---

## Cost Management & Optimization

### GPU Cost Optimization

```python
class GPUCostOptimizer:
    """
    Optimize GPU costs for AI inference
    """
    def __init__(self):
        self.gpu_instances = {}
        self.cost_tracker = CostTracker()
    
    def select_gpu_instance(self, model_config, request):
        """
        Select optimal GPU instance for request
        """
        requirements = {
            'memory': model_config.get('memory_gb', 8),
            'compute': model_config.get('compute_units', 1),
            'latency': request.get('max_latency_ms', 1000)
        }
        
        # Get available instances
        available = self.get_available_instances()
        
        # Score each instance
        scored = []
        for instance in available:
            score = self.score_instance(instance, requirements)
            cost = self.calculate_cost(instance, request)
            
            scored.append({
                'instance': instance,
                'score': score,
                'cost': cost,
                'value': score / cost  # Best value
            })
        
        # Sort by value
        scored.sort(key=lambda x: x['value'], reverse=True)
        
        return scored[0]['instance']
    
    def score_instance(self, instance, requirements):
        """Score instance based on requirements"""
        score = 0
        
        # Memory fit (don't overprovision)
        memory_ratio = instance['memory_gb'] / requirements['memory']
        if memory_ratio < 1:
            return 0  # Can't fit
        elif memory_ratio < 2:
            score += 50  # Good fit
        else:
            score += 20  # Overprovisioned
        
        # Compute capability
        if instance['compute_units'] >= requirements['compute']:
            score += 30
        
        # Latency capability
        if instance['typical_latency_ms'] <= requirements['latency']:
            score += 20
        
        return score
    
    def calculate_cost(self, instance, request):
        """Calculate cost for request"""
        # Cost per token
        cost_per_token = instance['cost_per_1k_tokens'] / 1000
        
        # Estimated tokens
        estimated_tokens = request.get('estimated_tokens', 1000)
        
        return cost_per_token * estimated_tokens
    
    def optimize_batching(self, requests, max_batch_size=32):
        """
        Batch requests to maximize GPU utilization
        """
        # Group requests by model
        by_model = {}
        for req in requests:
            model = req['model_name']
            if model not in by_model:
                by_model[model] = []
            by_model[model].append(req)
        
        # Batch each model's requests
        batches = []
        for model, model_requests in by_model.items():
            # Sort by latency requirement
            model_requests.sort(key=lambda x: x.get('max_latency_ms', float('inf')))
            
            # Create batches
            for i in range(0, len(model_requests), max_batch_size):
                batch = model_requests[i:i + max_batch_size]
                batches.append({
                    'model': model,
                    'requests': batch,
                    'size': len(batch)
                })
        
        return batches
```

### Token Cost Tracking

```python
class TokenCostTracker:
    """
    Track and optimize token costs
    """
    def __init__(self):
        self.pricing = {
            'gpt-4': {'input': 0.03, 'output': 0.06},
            'gpt-3.5-turbo': {'input': 0.0015, 'output': 0.002},
            'claude-3': {'input': 0.015, 'output': 0.075}
        }
        self.usage = {}
    
    def track_usage(self, model, input_tokens, output_tokens, user_id):
        """Track token usage"""
        if user_id not in self.usage:
            self.usage[user_id] = {
                'total_input_tokens': 0,
                'total_output_tokens': 0,
                'total_cost': 0.0,
                'requests': 0
            }
        
        # Calculate cost
        model_pricing = self.pricing.get(model, {'input': 0.01, 'output': 0.02})
        cost = (input_tokens * model_pricing['input'] + 
                output_tokens * model_pricing['output']) / 1000
        
        # Update usage
        self.usage[user_id]['total_input_tokens'] += input_tokens
        self.usage[user_id]['total_output_tokens'] += output_tokens
        self.usage[user_id]['total_cost'] += cost
        self.usage[user_id]['requests'] += 1
        
        return {
            'cost': cost,
            'total_cost': self.usage[user_id]['total_cost']
        }
    
    def get_cost_report(self, user_id, time_range='day'):
        """Get cost report for user"""
        user_usage = self.usage.get(user_id, {})
        
        return {
            'user_id': user_id,
            'time_range': time_range,
            'total_cost': user_usage.get('total_cost', 0.0),
            'total_requests': user_usage.get('requests', 0),
            'avg_cost_per_request': user_usage.get('total_cost', 0.0) / max(user_usage.get('requests', 1), 1),
            'input_tokens': user_usage.get('total_input_tokens', 0),
            'output_tokens': user_usage.get('total_output_tokens', 0)
        }
    
    def suggest_optimizations(self, user_id):
        """Suggest cost optimizations"""
        suggestions = []
        user_usage = self.usage.get(user_id, {})
        
        # Check for expensive models
        if user_usage.get('total_cost', 0) > 100:
            suggestions.append({
                'type': 'model_downgrade',
                'message': 'Consider using GPT-3.5-Turbo instead of GPT-4 for simple tasks',
                'potential_savings': '40-60%'
            })
        
        # Check for high token usage
        if user_usage.get('total_input_tokens', 0) > 1000000:
            suggestions.append({
                'type': 'context_optimization',
                'message': 'Optimize context to reduce input tokens',
                'potential_savings': '20-30%'
            })
        
        # Check for caching opportunities
        if user_usage.get('requests', 0) > 1000:
            suggestions.append({
                'type': 'caching',
                'message': 'Implement caching for frequent queries',
                'potential_savings': '30-50%'
            })
        
        return suggestions
```

### Cost Allocation

```python
class CostAllocator:
    """
    Allocate costs to teams/projects
    """
    def __init__(self):
        self.allocations = {}
    
    def track_cost(self, team, project, cost):
        """Track cost for team/project"""
        key = f"{team}:{project}"
        
        if key not in self.allocations:
            self.allocations[key] = {
                'team': team,
                'project': project,
                'total_cost': 0.0,
                'requests': 0,
                'daily_costs': {}
            }
        
        self.allocations[key]['total_cost'] += cost
        self.allocations[key]['requests'] += 1
        
        # Track daily
        today = datetime.now().date().isoformat()
        if today not in self.allocations[key]['daily_costs']:
            self.allocations[key]['daily_costs'][today] = 0.0
        self.allocations[key]['daily_costs'][today] += cost
    
    def get_cost_report(self, team=None, project=None):
        """Get cost report"""
        if team and project:
            key = f"{team}:{project}"
            return self.allocations.get(key, {})
        elif team:
            # Aggregate for team
            team_costs = {
                'team': team,
                'total_cost': 0.0,
                'projects': []
            }
            
            for key, alloc in self.allocations.items():
                if key.startswith(f"{team}:"):
                    team_costs['total_cost'] += alloc['total_cost']
                    team_costs['projects'].append(alloc)
            
            return team_costs
        else:
            # All costs
            return list(self.allocations.values())
    
    def get_cost_breakdown(self):
        """Get cost breakdown by team/project"""
        breakdown = []
        
        for key, alloc in self.allocations.items():
            breakdown.append({
                'team': alloc['team'],
                'project': alloc['project'],
                'total_cost': alloc['total_cost'],
                'requests': alloc['requests'],
                'avg_cost_per_request': alloc['total_cost'] / max(alloc['requests'], 1)
            })
        
        # Sort by cost
        breakdown.sort(key=lambda x: x['total_cost'], reverse=True)
        
        return breakdown
```

---

## Centralized vs. Federated Architecture

### Centralized Architecture

Single platform serving all teams.

```python
class CentralizedAIPlatform:
    """
    Centralized AI platform
    """
    def __init__(self):
        self.shared_inference_gateway = InferenceGateway()
        self.shared_model_registry = ModelRegistry()
        self.shared_feature_store = FeatureStore()
        self.shared_monitoring = MonitoringSystem()
    
    def serve_team(self, team_name, request):
        """Serve request from centralized platform"""
        # Route through shared gateway
        response = self.shared_inference_gateway.route_request(
            request['model_name'],
            request
        )
        
        # Track team usage
        self.cost_allocator.track_cost(team_name, 'shared', response['cost'])
        
        return response
```

**Pros:**
- Economies of scale
- Consistent tooling
- Centralized control
- Easier maintenance

**Cons:**
- Single point of failure
- Less flexibility
- Potential bottlenecks
- Team dependencies

### Federated Architecture

Each team has own AI infrastructure.

```python
class FederatedAIPlatform:
    """
    Federated AI platform
    """
    def __init__(self):
        self.team_platforms = {}
        self.shared_services = {
            'model_registry': ModelRegistry(),
            'monitoring': MonitoringSystem()
        }
    
    def register_team_platform(self, team_name, platform):
        """Register team's AI platform"""
        self.team_platforms[team_name] = platform
    
    def route_request(self, team_name, request):
        """Route to team's platform"""
        team_platform = self.team_platforms.get(team_name)
        
        if not team_platform:
            return {'error': 'Team platform not found'}
        
        # Route to team's gateway
        response = team_platform.inference_gateway.route_request(
            request['model_name'],
            request
        )
        
        # Track in shared monitoring
        self.shared_services['monitoring'].track_request(team_name, request, response)
        
        return response
```

**Pros:**
- Team autonomy
- Independent scaling
- No bottlenecks
- Technology flexibility

**Cons:**
- Duplication of effort
- Inconsistent tooling
- Higher total cost
- Harder governance

### Hybrid Architecture

```python
class HybridAIPlatform:
    """
    Hybrid: centralized for common services, federated for specialized
    """
    def __init__(self):
        # Shared services
        self.shared = {
            'model_registry': ModelRegistry(),
            'feature_store': FeatureStore(),
            'monitoring': MonitoringSystem(),
            'cost_tracking': CostAllocator()
        }
        
        # Team-specific inference
        self.team_gateways = {}
    
    def get_inference(self, team_name, model_name, request):
        """Get inference from appropriate gateway"""
        # Check if team has own gateway
        if team_name in self.team_gateways:
            # Use team's gateway
            gateway = self.team_gateways[team_name]
        else:
            # Use shared gateway
            gateway = self.shared_inference_gateway
        
        # Execute
        response = gateway.route_request(model_name, request)
        
        # Track costs
        self.shared['cost_tracking'].track_cost(
            team_name,
            model_name,
            response.get('cost', 0)
        )
        
        return response
```

### Decision Framework

| Aspect | Centralized | Federated | Hybrid |
|--------|-------------|-----------|--------|
| **Cost** | Lower (economies of scale) | Higher (duplication) | Medium |
| **Flexibility** | Low | High | Medium-High |
| **Scalability** | Potential bottlenecks | Independent scaling | Best of both |
| **Control** | High | Low | Medium |
| **Team Autonomy** | Low | High | Medium |
| **Best For** | Small orgs, standard needs | Large orgs, diverse needs | Most enterprises |

**Recommendation:**
- **Start centralized** for common models and services
- **Allow federation** for specialized, high-volume use cases
- **Shared registry and monitoring** across all deployments

---

## Observability & Monitoring

### Three Pillars of Observability

```python
class AIObservability:
    """
    Comprehensive observability for AI systems
    """
    def __init__(self):
        self.metrics = MetricsCollector()
        self.logs = LogCollector()
        self.traces = TraceCollector()
    
    def record_inference(self, model_name, request, response, latency):
        """Record inference metrics"""
        # Metrics
        self.metrics.record({
            'model': model_name,
            'latency_ms': latency,
            'input_tokens': request.get('tokens', 0),
            'output_tokens': response.get('tokens', 0),
            'success': response.get('success', False),
            'cost': self.calculate_cost(request, response)
        })
        
        # Logs
        self.logs.record({
            'level': 'info',
            'model': model_name,
            'request_id': request.get('id'),
            'message': f"Inference completed in {latency}ms"
        })
        
        # Traces
        trace_id = request.get('trace_id')
        self.traces.add_event(trace_id, 'inference_complete', {
            'model': model_name,
            'latency': latency,
            'tokens': response.get('tokens', 0)
        })
```

### Key Metrics

```python
class AIMetrics:
    """Key metrics for AI systems"""
    
    @staticmethod
    def get_system_metrics():
        """System-level metrics"""
        return {
            'latency': {
                'p50': 'Target: <500ms',
                'p95': 'Target: <1000ms',
                'p99': 'Target: <2000ms'
            },
            'throughput': {
                'requests_per_second': 'Target: >100',
                'tokens_per_second': 'Target: >10000'
            },
            'availability': {
                'uptime': 'Target: 99.9%',
                'error_rate': 'Target: <0.1%'
            },
            'resource_utilization': {
                'gpu_utilization': 'Target: 70-80%',
                'memory_utilization': 'Target: <90%'
            }
        }
    
    @staticmethod
    def get_model_metrics():
        """Model-level metrics"""
        return {
            'quality': {
                'accuracy': 'Model accuracy on test set',
                'precision': 'Retrieval precision',
                'recall': 'Retrieval recall',
                'f1_score': 'F1 score'
            },
            'performance': {
                'inference_time': 'Time per inference',
                'queue_time': 'Time waiting in queue',
                'total_latency': 'End-to-end latency'
            },
            'usage': {
                'requests_per_minute': 'RPM',
                'tokens_per_minute': 'TPM',
                'unique_users': 'Active users'
            },
            'cost': {
                'cost_per_request': 'Average cost',
                'cost_per_1k_tokens': 'Token cost',
                'daily_cost': 'Daily spend'
            }
        }
    
    @staticmethod
    def get_business_metrics():
        """Business-level metrics"""
        return {
            'adoption': {
                'daily_active_users': 'DAU',
                'weekly_active_users': 'WAU',
                'monthly_active_users': 'MAU'
            },
            'satisfaction': {
                'csat_score': 'Customer satisfaction',
                'nps': 'Net promoter score',
                'escalation_rate': 'Human escalation rate'
            },
            'outcomes': {
                'task_completion_rate': 'Tasks completed',
                'time_saved': 'Hours saved per user',
                'error_reduction': 'Errors prevented'
            }
        }
```

### Monitoring Architecture

```python
class MonitoringArchitecture:
    """
    Complete monitoring architecture
    """
    def __init__(self):
        self.metrics_collector = MetricsCollector()
        self.log_aggregator = LogAggregator()
        self.distributed_tracer = DistributedTracer()
        self.alert_manager = AlertManager()
        self.dashboard = Dashboard()
    
    def setup_monitoring(self, component):
        """Setup monitoring for component"""
        # Metrics
        self.metrics_collector.register_component(component)
        
        # Logs
        self.log_aggregator.add_source(component.log_source)
        
        # Traces
        self.distributed_tracer.register_service(component.service_name)
        
        # Alerts
        self.alert_manager.add_rules(component.alert_rules)
    
    def create_dashboard(self, name, queries):
        """Create monitoring dashboard"""
        panels = []
        
        for query in queries:
            panel = {
                'title': query['title'],
                'query': query['query'],
                'visualization': query.get('type', 'graph')
            }
            panels.append(panel)
        
        self.dashboard.create(name, panels)
    
    def setup_alerts(self, rules):
        """Setup alerting rules"""
        for rule in rules:
            self.alert_manager.add_rule({
                'name': rule['name'],
                'condition': rule['condition'],
                'threshold': rule['threshold'],
                'duration': rule.get('duration', '5m'),
                'severity': rule['severity'],
                'notification': rule['notification']
            })
```

### Alerting Strategy

```python
class AlertingStrategy:
    """Define alerting rules for AI systems"""
    
    @staticmethod
    def get_default_alerts():
        """Default alerting rules"""
        return [
            {
                'name': 'high_latency',
                'condition': 'p95_latency > 2000ms',
                'severity': 'warning',
                'notification': 'team_channel'
            },
            {
                'name': 'high_error_rate',
                'condition': 'error_rate > 1%',
                'severity': 'critical',
                'notification': 'oncall'
            },
            {
                'name': 'model_degradation',
                'condition': 'accuracy_drop > 5%',
                'severity': 'warning',
                'notification': 'ml_team'
            },
            {
                'name': 'cost_spike',
                'condition': 'hourly_cost > 2x_baseline',
                'severity': 'warning',
                'notification': 'finance_team'
            },
            {
                'name': 'queue_backlog',
                'condition': 'queue_length > 1000',
                'severity': 'critical',
                'notification': 'platform_team'
            }
        ]
```

---

## Scalability & Performance

### Scaling Strategies

#### Vertical Scaling (Scale Up)

```python
class VerticalScaler:
    """Scale up individual instances"""
    def __init__(self):
        self.instance_types = {
            'small': {'gpu': 'T4', 'memory': '16GB', 'cost': 0.5},
            'medium': {'gpu': 'A10G', 'memory': '32GB', 'cost': 2.0},
            'large': {'gpu': 'A100', 'memory': '64GB', 'cost': 8.0},
            'xlarge': {'gpu': 'A100-80GB', 'memory': '128GB', 'cost': 16.0}
        }
    
    def recommend_instance(self, model_config, request):
        """Recommend instance type"""
        # Calculate requirements
        memory_needed = model_config.get('memory_gb', 8)
        compute_needed = model_config.get('gpu_compute', 1)
        latency_requirement = request.get('max_latency_ms', 1000)
        
        # Find suitable instance
        for instance_name, specs in self.instance_types.items():
            if specs['memory'] >= memory_needed and specs['compute'] >= compute_needed:
                return {
                    'instance_type': instance_name,
                    'specs': specs,
                    'estimated_latency': self.estimate_latency(instance_name, model_config),
                    'cost_per_hour': specs['cost']
                }
        
        return {'instance_type': 'xlarge', 'specs': self.instance_types['xlarge']}
```

#### Horizontal Scaling (Scale Out)

```python
class HorizontalScaler:
    """Scale out with multiple instances"""
    def __init__(self):
        self.instances = {}
        self.load_balancer = LoadBalancer()
        self.auto_scaler = AutoScaler()
    
    def scale_out(self, model_name, target_instances):
        """Scale out to target number of instances"""
        current_instances = len(self.instances.get(model_name, []))
        
        if target_instances > current_instances:
            # Scale up
            for i in range(current_instances, target_instances):
                instance = self.create_instance(model_name)
                self.instances[model_name].append(instance)
                self.load_balancer.add_endpoint(model_name, instance.endpoint)
        
        elif target_instances < current_instances:
            # Scale down
            excess = current_instances - target_instances
            for i in range(excess):
                instance = self.instances[model_name].pop()
                self.terminate_instance(instance)
                self.load_balancer.remove_endpoint(model_name, instance.endpoint)
    
    def auto_scale(self, model_name):
        """Auto-scale based on metrics"""
        # Get current metrics
        metrics = self.get_model_metrics(model_name)
        
        # Calculate target instances
        current_rps = metrics['requests_per_second']
        target_rps = self.calculate_target_rps(metrics)
        
        # Calculate instances needed
        instances_per_rps = 10  # 1 instance can handle 10 RPS
        target_instances = max(1, int(target_rps / instances_per_rps))
        
        # Scale
        self.scale_out(model_name, target_instances)
    
    def calculate_target_rps(self, metrics):
        """Calculate target RPS based on metrics"""
        # Factor in queue length, latency, etc.
        queue_length = metrics.get('queue_length', 0)
        current_latency = metrics.get('p95_latency_ms', 0)
        
        # If queue building or latency high, scale up
        if queue_length > 100 or current_latency > 1000:
            return metrics['requests_per_second'] * 1.5
        
        return metrics['requests_per_second']
```

### Performance Optimization

```python
class PerformanceOptimizer:
    """Optimize inference performance"""
    def __init__(self):
        self.optimizations = {
            'batching': self.optimize_batching,
            'caching': self.optimize_caching,
            'model_compression': self.optimize_model,
            'parallelism': self.optimize_parallelism
        }
    
    def optimize_batching(self, requests):
        """
        Batch requests for efficiency
        """
        # Group by model
        batches = {}
        for req in requests:
            model = req['model_name']
            if model not in batches:
                batches[model] = []
            batches[model].append(req)
        
        # Create batches
        optimized = []
        for model, model_requests in batches.items():
            # Batch size based on model
            batch_size = self.get_optimal_batch_size(model)
            
            for i in range(0, len(model_requests), batch_size):
                batch = model_requests[i:i + batch_size]
                optimized.append({
                    'type': 'batch',
                    'model': model,
                    'requests': batch
                })
        
        return optimized
    
    def optimize_caching(self, request):
        """
        Optimize caching strategy
        """
        # Check if request is cacheable
        if not self.is_cacheable(request):
            return None
        
        # Generate cache key
        cache_key = self.generate_cache_key(request)
        
        # Check cache
        cached = self.cache.get(cache_key)
        if cached:
            return cached
        
        return None
    
    def optimize_model(self, model_config):
        """
        Optimize model for inference
        """
        optimizations = []
        
        # Quantization
        if model_config.get('precision') == 'fp32':
            optimizations.append({
                'type': 'quantization',
                'from': 'fp32',
                'to': 'fp16',
                'speedup': '2x',
                'memory_savings': '50%'
            })
        
        # Pruning
        if model_config.get('size_mb', 0) > 1000:
            optimizations.append({
                'type': 'pruning',
                'sparsity': 0.5,
                'speedup': '1.5x',
                'memory_savings': '50%'
            })
        
        return optimizations
    
    def optimize_parallelism(self, model_config):
        """
        Optimize model parallelism
        """
        model_size_gb = model_config.get('size_gb', 1)
        
        if model_size_gb > 20:
            # Use tensor parallelism
            return {
                'strategy': 'tensor_parallelism',
                'num_gpus': 4,
                'reason': 'Model too large for single GPU'
            }
        elif model_size_gb > 10:
            # Use pipeline parallelism
            return {
                'strategy': 'pipeline_parallelism',
                'num_gpus': 2,
                'reason': 'Medium-sized model'
            }
        else:
            # Single GPU
            return {
                'strategy': 'single_gpu',
                'num_gpus': 1,
                'reason': 'Model fits on single GPU'
            }
```

---

## Security & Compliance

### Security Architecture

```python
class AISecurity:
    """
    Security controls for AI platforms
    """
    def __init__(self):
        self.auth = Authentication()
        self.authorization = Authorization()
        self.encryption = Encryption()
        self.audit_log = AuditLog()
    
    def authenticate_request(self, request):
        """Authenticate request"""
        # Extract credentials
        api_key = request.get('api_key')
        token = request.get('token')
        
        # Validate
        if api_key:
            return self.auth.validate_api_key(api_key)
        elif token:
            return self.auth.validate_token(token)
        
        return {'authenticated': False, 'error': 'No credentials'}
    
    def authorize_request(self, user, model_name, action):
        """Authorize request"""
        # Check permissions
        permissions = self.authorization.get_permissions(user)
        
        if action not in permissions.get('allowed_actions', []):
            return {'authorized': False, 'error': 'Permission denied'}
        
        if model_name not in permissions.get('allowed_models', []):
            return {'authorized': False, 'error': 'Model access denied'}
        
        return {'authorized': True}
    
    def encrypt_data(self, data, key):
        """Encrypt sensitive data"""
        return self.encryption.encrypt(data, key)
    
    def audit_request(self, user, action, details):
        """Audit request"""
        self.audit_log.record({
            'user': user,
            'action': action,
            'details': details,
            'timestamp': datetime.now().isoformat()
        })
```

### PII Handling

```python
class PIIHandler:
    """Handle PII in AI requests/responses"""
    def __init__(self):
        self.pii_detector = PIIDetector()
        self.pii_redactor = PIIRedactor()
    
    def process_request(self, request):
        """Process request for PII"""
        # Detect PII
        pii_found = self.pii_detector.detect(request)
        
        if pii_found:
            # Redact PII
            redacted_request = self.pii_redactor.redact(request)
            
            return {
                'processed': redacted_request,
                'pii_detected': True,
                'pii_types': [p['type'] for p in pii_found]
            }
        
        return {
            'processed': request,
            'pii_detected': False
        }
    
    def process_response(self, response):
        """Process response for PII leakage"""
        # Check if response contains PII
        pii_found = self.pii_detector.detect(response)
        
        if pii_found:
            # Block response
            return {
                'blocked': True,
                'error': 'Response contains sensitive information'
            }
        
        return {'blocked': False, 'response': response}
```

### Compliance Framework

```python
class ComplianceFramework:
    """Ensure compliance with regulations"""
    def __init__(self):
        self.regulations = {
            'GDPR': self.check_gdpr_compliance,
            'HIPAA': self.check_hipaa_compliance,
            'SOC2': self.check_soc2_compliance
        }
    
    def check_compliance(self, regulation, operation):
        """Check if operation complies with regulation"""
        checker = self.regulations.get(regulation)
        if checker:
            return checker(operation)
        return {'compliant': True}
    
    def check_gdpr_compliance(self, operation):
        """Check GDPR compliance"""
        checks = {
            'data_minimization': self.check_data_minimization(operation),
            'consent': self.check_consent(operation),
            'right_to_erasure': self.check_erasure_capability(operation)
        }
        
        return {
            'regulation': 'GDPR',
            'compliant': all(checks.values()),
            'checks': checks
        }
    
    def check_hipaa_compliance(self, operation):
        """Check HIPAA compliance"""
        checks = {
            'encryption': self.check_encryption(operation),
            'access_control': self.check_access_control(operation),
            'audit_logging': self.check_audit_logging(operation)
        }
        
        return {
            'regulation': 'HIPAA',
            'compliant': all(checks.values()),
            'checks': checks
        }
```

---

## Hands-On Exercises

### Exercise 1: Design an Inference Gateway

**Objective:** Design and implement a basic inference gateway

**Task:**
1. Create gateway with routing logic
2. Add load balancing
3. Implement caching
4. Add rate limiting

**Solution:**

```python
class BasicInferenceGateway:
    def __init__(self):
        self.models = {}
        self.cache = {}
        self.rate_limiter = RateLimiter()
    
    def register_model(self, name, endpoint):
        self.models[name] = {
            'endpoint': endpoint,
            'healthy': True
        }
    
    def infer(self, model_name, request, user_id):
        # Rate limit
        if not self.rate_limiter.allow(user_id):
            return {'error': 'Rate limit exceeded'}
        
        # Check cache
        cache_key = f"{model_name}:{hash(request)}"
        if cache_key in self.cache:
            return self.cache[cache_key]
        
        # Route to model
        endpoint = self.models[model_name]['endpoint']
        response = requests.post(endpoint, json=request)
        
        # Cache response
        self.cache[cache_key] = response
        
        return response
```

### Exercise 2: Implement Cost Tracking

**Objective:** Build cost tracking and optimization system

**Solution:**

```python
class SimpleCostTracker:
    def __init__(self):
        self.pricing = {
            'gpt-4': {'input': 0.03, 'output': 0.06},
            'gpt-3.5': {'input': 0.0015, 'output': 0.002}
        }
        self.usage = {}
    
    def track(self, user_id, model, input_tokens, output_tokens):
        cost = (input_tokens * self.pricing[model]['input'] + 
                output_tokens * self.pricing[model]['output']) / 1000
        
        if user_id not in self.usage:
            self.usage[user_id] = {'cost': 0, 'requests': 0}
        
        self.usage[user_id]['cost'] += cost
        self.usage[user_id]['requests'] += 1
        
        return cost
    
    def get_report(self, user_id):
        return self.usage.get(user_id, {})
```

### Exercise 3: Design Monitoring Dashboard

**Objective:** Create monitoring dashboard for AI platform

**Solution:**

```python
class MonitoringDashboard:
    def __init__(self):
        self.metrics = {}
    
    def record_metric(self, model, metric_name, value):
        if model not in self.metrics:
            self.metrics[model] = {}
        
        if metric_name not in self.metrics[model]:
            self.metrics[model][metric_name] = []
        
        self.metrics[model][metric_name].append({
            'value': value,
            'timestamp': datetime.now()
        })
    
    def get_dashboard_data(self, model):
        return {
            'latency': self.calculate_percentiles(self.metrics[model].get('latency', [])),
            'throughput': len(self.metrics[model].get('requests', [])),
            'error_rate': self.calculate_error_rate(self.metrics[model].get('errors', [])),
            'cost': self.calculate_cost(self.metrics[model].get('tokens', []))
        }
```

### Exercise 4: Implement Auto-Scaling

**Objective:** Build auto-scaling for inference

**Solution:**

```python
class AutoScaler:
    def __init__(self):
        self.instances = {}
        self.metrics_collector = MetricsCollector()
    
    def should_scale(self, model_name):
        metrics = self.metrics_collector.get_metrics(model_name)
        
        # Scale up if queue length > 100 or latency > 1000ms
        if metrics['queue_length'] > 100 or metrics['p95_latency'] > 1000:
            return 'scale_up'
        
        # Scale down if utilization < 30%
        if metrics['gpu_utilization'] < 30:
            return 'scale_down'
        
        return 'no_change'
    
    def scale(self, model_name, action):
        current_instances = len(self.instances.get(model_name, []))
        
        if action == 'scale_up':
            target = current_instances + 1
        elif action == 'scale_down':
            target = max(1, current_instances - 1)
        else:
            return
        
        self.adjust_instances(model_name, target)
```

### Exercise 5: Design Security Layer

**Objective:** Implement security controls

**Solution:**

```python
class SecurityLayer:
    def __init__(self):
        self.api_keys = {}
        self.rate_limiter = RateLimiter()
        self.pii_detector = PIIDetector()
    
    def validate_request(self, request):
        # Check API key
        api_key = request.get('api_key')
        if not self.validate_api_key(api_key):
            return {'allowed': False, 'error': 'Invalid API key'}
        
        # Check rate limit
        if not self.rate_limiter.allow(api_key):
            return {'allowed': False, 'error': 'Rate limit exceeded'}
        
        # Check PII
        pii = self.pii_detector.detect(request)
        if pii:
            return {'allowed': False, 'error': 'PII detected'}
        
        return {'allowed': True}
```

---

## Practice Question Bank

### Multiple Choice Questions

**1. What is the primary purpose of an inference gateway?**
A) Train models  
B) Route and manage inference requests  
C) Store data  
D) Monitor costs  

**Answer: B**  
**Explanation:** Inference gateway routes, load balances, and manages inference requests to models.

---

**2. What is the benefit of batching in inference?**
A) Lower accuracy  
B) Higher GPU utilization and lower cost  
C) Slower inference  
D) More complex code  

**Answer: B**  
**Explanation:** Batching increases GPU utilization, reducing cost per inference.

---

**3. When should you use centralized vs. federated architecture?**
A) Always centralized  
B) Always federated  
C) Centralized for common services, federated for specialized needs  
D) Doesn't matter  

**Answer: C**  
**Explanation:** Hybrid approach: centralized for common services (cost savings), federated for specialized needs (flexibility).

---

**4. What is the primary benefit of observability?**
A) Lower costs  
B) Faster debugging and proactive issue detection  
C) Better models  
D) Simpler code  

**Answer: B**  
**Explanation:** Observability (metrics, logs, traces) enables faster debugging and proactive detection of issues.

---

**5. What is the best way to reduce inference costs?**
A) Use smaller models  
B) Optimize batching, caching, and model selection  
C) Reduce quality  
D) Use fewer GPUs  

**Answer: B**  
**Explanation:** Multi-faceted approach: batching increases utilization, caching reduces redundant calls, model selection balances cost/quality.

---

**6. What should you monitor in production AI systems?**
A) Only latency  
B) Latency, errors, costs, quality, and business metrics  
C) Only costs  
D) Only model accuracy  

**Answer: B**  
**Explanation:** Comprehensive monitoring includes system metrics (latency, errors), model metrics (accuracy), and business metrics (CSAT).

---

**7. What is the purpose of circuit breakers in inference gateways?**
A) Stop all traffic  
B) Prevent cascading failures when model is down  
C) Reduce costs  
D) Improve latency  

**Answer: B**  
**Explanation:** Circuit breakers stop sending requests to failing models, preventing cascading failures.

---

**8. How do you handle PII in AI systems?**
A) Ignore it  
B) Detect, redact, or block PII based on policy  
C) Store it all  
D) Only in logs  

**Answer: B**  
**Explanation:** PII handling requires detection and policy-based action (redact, block) to ensure compliance.

---

**9. What is the benefit of a model registry?**
A) Train models faster  
B) Centralized version control and metadata for models  
C) Reduce costs  
D) Improve accuracy  

**Answer: B**  
**Explanation:** Model registry provides version control, metadata management, and deployment tracking.

---

**10. When should you auto-scale AI infrastructure?**
A) Never  
B) Always  
C) When metrics indicate need (queue length, latency, utilization)  
D) Only during business hours  

**Answer: C**  
**Explanation:** Auto-scale based on metrics: scale up when queue/latency high, scale down when utilization low.

---

### Scenario-Based Questions

**11. Scenario:** Your inference costs spiked 3x this month. What's the issue?

A) Model is broken  
B) Need to optimize batching, implement caching, review model selection  
C) Need more GPUs  
D) Users are abusing system  

**Answer: B**  
**Explanation:** Cost spikes typically from poor batching, no caching, or wrong model selection. Optimize these first.

---

**12. Scenario:** You need to support 10K concurrent users with <100ms latency. What architecture?

A) Single GPU  
B) Distributed inference gateway with load balancing and caching  
C) Centralized platform  
D) Federated architecture  

**Answer: B**  
**Explanation:** High concurrency + low latency requires distributed architecture with load balancing and caching.

---

**13. Scenario:** Different teams need different model versions. How to manage?

A) Force all teams to use same version  
B) Model registry with versioning and team-specific deployments  
C) Separate platforms per team  
D) Manual coordination  

**Answer: B**  
**Explanation:** Model registry with versioning allows teams to use different versions while maintaining central control.

---

**14. Scenario:** Your platform needs to comply with GDPR. What to implement?

A) Nothing special  
B) Data minimization, consent management, right to erasure  
C) Only encryption  
D) Only access controls  

**Answer: B**  
**Explanation:** GDPR requires data minimization, consent, and right to erasure (ability to delete user data).

---

**15. Scenario:** Model accuracy dropped 10% after deployment. What happened?

A) Model is broken  
B) Data drift - need monitoring and retraining pipeline  
C) Need better hardware  
D) Code bug  

**Answer: B**  
**Explanation:** Accuracy drops typically indicate data drift. Need monitoring to detect and retraining pipeline to fix.

---

### True/False Questions

**16. Inference costs are usually lower than training costs.**  
**Answer: False**  
**Explanation:** Inference costs often exceed training costs over time, especially for high-traffic models.

---

**17. Centralized platforms are always cheaper than federated.**  
**Answer: False**  
**Explanation:** While centralized has economies of scale, federated can be cheaper for specialized, high-volume use cases.

---

**18. You should always use the largest GPU for best performance.**  
**Answer: False**  
**Explanation:** Right-size GPUs based on requirements. Overprovisioning wastes money without benefit.

---

**19. Observability is only about monitoring metrics.**  
**Answer: False**  
**Explanation:** Observability includes metrics, logs, and traces - all three pillars are needed.

---

**20. Auto-scaling should be based on fixed schedules only.**  
**Answer: False**  
**Explanation:** Auto-scaling should be metrics-driven (queue length, latency, utilization) for optimal results.

---

### Short Answer Questions

**21. Design an inference gateway architecture for 100K daily active users.**

**Answer:**

1. **Gateway Layer:**
   - Multiple gateway instances behind load balancer
   - API gateway for authentication and rate limiting
   - CDN for static assets

2. **Caching Layer:**
   - Redis for frequent queries
   - Cache hit ratio target: 30-50%
   - TTL based on content type

3. **Model Serving:**
   - Model registry for version management
   - Auto-scaling based on queue length
   - GPU instances (A100/A10G) for inference

4. **Monitoring:**
   - Latency (p50, p95, p99)
   - Error rates
   - Cost per request
   - Model accuracy

5. **Cost Optimization:**
   - Batching for throughput
   - Model quantization
   - Spot instances for non-critical workloads

---

**22. How do you optimize GPU costs for AI inference?**

**Answer:**

1. **Right-sizing:** Match GPU to model size (don't overprovision)
2. **Batching:** Maximize GPU utilization with request batching
3. **Caching:** Cache frequent queries to avoid redundant inference
4. **Model Optimization:** Quantization, pruning, distillation
5. **Auto-scaling:** Scale based on demand, not fixed capacity
6. **Spot Instances:** Use spot/preemptible instances for fault-tolerant workloads
7. **Model Selection:** Use smaller models when possible (GPT-3.5 vs GPT-4)
8. **Monitoring:** Track cost per request and set alerts

---

**23. Design a monitoring strategy for production AI systems.**

**Answer:**

**System Metrics:**
- Latency (p50, p95, p99)
- Throughput (RPS, TPS)
- Error rate
- Availability

**Model Metrics:**
- Inference time
- Queue length
- Model accuracy
- Drift detection

**Business Metrics:**
- User satisfaction (CSAT)
- Task completion rate
- Cost per user
- Escalation rate

**Alerting:**
- Latency > 2s (p95)
- Error rate > 1%
- Cost spike > 2x baseline
- Model accuracy drop > 5%

**Dashboards:**
- Real-time: Latency, errors, throughput
- Daily: Cost, usage patterns
- Weekly: Model performance, business impact

---

**24. Compare centralized vs. federated AI platforms.**

**Answer:**

**Centralized:**
- Pros: Economies of scale, consistent tooling, easier governance
- Cons: Single point of failure, less flexibility, bottlenecks
- Best for: Small-medium orgs, standard use cases

**Federated:**
- Pros: Team autonomy, independent scaling, technology flexibility
- Cons: Duplication, inconsistent tooling, higher total cost
- Best for: Large orgs, diverse specialized needs

**Hybrid (Recommended):**
- Centralized: Common services (model registry, monitoring, feature store)
- Federated: Team-specific inference and specialized models
- Best of both worlds

---

**25. Design a security and compliance framework for AI platforms.**

**Answer:**

**Authentication & Authorization:**
- API keys, OAuth, mTLS
- Role-based access control (RBAC)
- Model-level permissions

**Data Protection:**
- Encryption at rest and in transit
- PII detection and redaction
- Data retention policies

**Audit & Compliance:**
- Comprehensive audit logging
- GDPR compliance (data minimization, consent, erasure)
- HIPAA compliance (if healthcare)
- SOC2 Type II

**Monitoring:**
- Security event monitoring
- Anomaly detection
- Regular security scans

**Governance:**
- Model approval process
- Change management
- Regular compliance audits

---

## Self-Assessment Checklist

### Core Concepts

- [ ] I understand AI platform architecture and components
- [ ] I can design inference gateways
- [ ] I know how to optimize GPU costs
- [ ] I can decide between centralized and federated architectures
- [ ] I understand observability (metrics, logs, traces)
- [ ] I can design monitoring and alerting
- [ ] I know scalability patterns (vertical vs. horizontal)
- [ ] I can implement security and compliance controls

### Practical Skills

- [ ] I can build an inference gateway
- [ ] I can implement cost tracking
- [ ] I can optimize batching and caching
- [ ] I can design monitoring dashboards
- [ ] I can implement auto-scaling
- [ ] I can add security controls
- [ ] I can handle PII
- [ ] I can ensure compliance

### System Design

- [ ] I can design production AI platform architecture
- [ ] I can plan for scalability
- [ ] I can design for cost optimization
- [ ] I can plan for security and compliance
- [ ] I can design observability strategy
- [ ] I can choose between centralized and federated

### Knowledge Check

Score yourself (5 = expert, 3 = proficient, 1 = beginner):

1. Platform architecture: ___/5
2. Inference gateways: ___/5
3. Cost optimization: ___/5
4. Centralized vs. federated: ___/5
5. Observability: ___/5
6. Scalability: ___/5
7. Security & compliance: ___/5
8. Performance optimization: ___/5

**Overall Score:** ___/40

**Interpretation:**
- 32-40: Ready to move to Week 5
- 24-31: Review weak areas before proceeding
- <24: Re-study Week 4 materials

---

## Summary & Key Takeaways

### Week 4 in 60 Seconds

**AI Platforms & Infrastructure** provide the foundation for production AI systems.

**Key Principles:**
1. **Inference Gateway:** Central entry point for model serving
2. **Cost Optimization:** Batching, caching, right-sizing
3. **Architecture:** Centralized for common, federated for specialized
4. **Observability:** Metrics, logs, traces for all components
5. **Scalability:** Horizontal scaling with auto-scaling
6. **Security:** Authentication, authorization, PII handling, compliance

**Critical Insights:**
✅ Inference costs exceed training costs over time
✅ Right architecture depends on organization size and needs
✅ Observability is non-negotiable for production
✅ Cost optimization is continuous process
✅ Security and compliance must be built-in

### Looking Ahead to Week 5

Next week: **AI Operational Excellence** - evals, trust, reliability, and the capstone project.

**Homework:** Outline an internal AI platform strategy for a mid-sized organization. Detail workload routing, observability signals, and centralized vs. federated decisions.

---

## Further Reading

### Essential Reading

1. **"Designing Machine Learning Systems"** - Chip Huyen
   - Why: Comprehensive guide to ML system design

2. **"Machine Learning Engineering"** - Andrey Buylov
   - Why: Production ML best practices

3. **"Kubernetes for Machine Learning"**
   - Link: https://kubernetes.io/docs/tasks/
   - Why: Orchestrating ML workloads

### Tools & Frameworks

**Inference:**
- **TorchServe** - PyTorch model serving
- **TensorFlow Serving** - TensorFlow model serving
- **Triton Inference Server** - Multi-framework serving
- **BentoML** - Model serving framework

**Orchestration:**
- **Kubernetes** - Container orchestration
- **Docker** - Containerization
- **Helm** - Kubernetes package manager

**Monitoring:**
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Datadog** - APM and monitoring
- **LangSmith** - LLM observability

**Cost Management:**
- **Kubecost** - Kubernetes cost tracking
- **OpenCost** - Open-source cost monitoring

---

**🎯 Week 4 Complete! You now understand AI platforms and infrastructure.**

**➡️ Next:** [Week 5 - AI Operational Excellence & Capstone](Week-05-Operational-Excellence-Capstone-Complete-Guide.md)

---

*Estimated Reading Time:* 4-5 hours  
*Exercises Completion Time:* 4-5 hours  
*Total Time:* 10-12 hours