# MRPOne Domain Models

## 1. User and Access Management Models

### User Model
```python
class User:
    id: UUID
    username: str
    email: str
    password_hash: str
    role: UserRole  # ENUM: ADMIN, PRODUCTION_MANAGER, SHOP_FLOOR_SUPERVISOR, etc.
    department: Department
    permissions: List[Permission]
    last_login: DateTime
    is_active: bool
```

### Role and Permission Model
```python
class UserRole:
    id: UUID
    name: str  # e.g., "Production Manager"
    permissions: List[Permission]

class Permission:
    id: UUID
    code: str  # e.g., "VIEW_PRODUCTION_ORDERS", "EDIT_INVENTORY"
    description: str
```

## 2. Resource Management Models

### Equipment/Machine Model
```python
class Equipment:
    id: UUID
    name: str
    model_number: str
    manufacturer: str
    purchase_date: Date
    maintenance_history: List[MaintenanceRecord]
    current_status: EquipmentStatus  # ENUM: OPERATIONAL, MAINTENANCE, IDLE
    capacity: float
    efficiency_rating: float
    last_maintenance_date: DateTime
    expected_maintenance_interval: TimeDelta
```

### Worker Model
```python
class Worker:
    id: UUID
    name: str
    department: Department
    skills: List[Skill]
    current_assignment: ProductionOrder
    skill_matrix: Dict[Skill, ProficiencyLevel]
    training_history: List[TrainingRecord]
    productivity_metrics: WorkerPerformance
```

## 3. Inventory Management Models

### Product Model
```python
class Product:
    id: UUID
    name: str
    sku: str
    description: str
    bill_of_materials: List[BOMItem]
    manufacturing_process: List[ProductionStep]
    current_inventory_level: float
    minimum_stock_level: float
    maximum_stock_level: float
    lead_time: TimeDelta
    unit_cost: Decimal
```

### Inventory Item Model
```python
class InventoryItem:
    id: UUID
    product: Product
    quantity: float
    location: WarehouseLocation
    batch_number: str
    received_date: DateTime
    expiry_date: DateTime
    quality_status: QualityStatus
    supplier: Supplier
```

## 4. Production Planning Models

### Production Order Model
```python
class ProductionOrder:
    id: UUID
    product: Product
    quantity: float
    status: ProductionStatus  # ENUM: PLANNED, IN_PROGRESS, COMPLETED, CANCELLED
    start_date: DateTime
    expected_completion_date: DateTime
    actual_completion_date: DateTime
    assigned_equipment: List[Equipment]
    assigned_workers: List[Worker]
    production_steps: List[ProductionStep]
    quality_checks: List[QualityCheck]
    cost_tracking: CostBreakdown
```

### Production Step Model
```python
class ProductionStep:
    id: UUID
    name: str
    description: str
    required_skills: List[Skill]
    estimated_duration: TimeDelta
    required_equipment: List[Equipment]
    input_materials: List[InventoryItem]
    output_products: List[Product]
```

## 5. Supply Chain Models

### Supplier Model
```python
class Supplier:
    id: UUID
    name: str
    contact_information: ContactDetails
    rating: float
    reliability_score: float
    typical_lead_time: TimeDelta
    quality_performance: Dict[Product, QualityMetrics]
    pricing_history: List[PricingRecord]
```

### Purchase Order Model
```python
class PurchaseOrder:
    id: UUID
    supplier: Supplier
    items: List[PurchaseOrderItem]
    status: PurchaseOrderStatus
    order_date: DateTime
    expected_delivery_date: DateTime
    actual_delivery_date: DateTime
    total_cost: Decimal
    payment_terms: PaymentTerms
```

## 6. Quality Management Models

### Quality Check Model
```python
class QualityCheck:
    id: UUID
    production_order: ProductionOrder
    check_type: QualityCheckType
    inspector: Worker
    timestamp: DateTime
    status: QualityStatus
    measurements: Dict[str, Any]
    defects: List[Defect]
```

### Defect Model
```python
class Defect:
    id: UUID
    type: DefectType
    severity: DefectSeverity
    description: str
    detected_at: ProductionStep
    root_cause_analysis: str
    corrective_action: str
```

## 7. Analytics and Reporting Models

### Performance Metrics Model
```python
class PerformanceMetrics:
    production_efficiency: float
    equipment_utilization: float
    worker_productivity: float
    quality_rate: float
    cost_per_unit: Decimal
    lead_time: TimeDelta
    inventory_turnover: float
```

## 8. Workstation Sequence Model

### WorkstationSequence Class
```python
class WorkstationSequence:
    def __init__(self, product_id, sequence):
        """
        Represents a predefined manufacturing sequence for a specific product
        
        Args:
            product_id (str): Unique identifier for the product
            sequence (List[Workstation]): Ordered list of workstations
        """
        self.product_id = product_id
        self.sequence = sequence
        self.dependencies = self._analyze_dependencies()
    
    def _analyze_dependencies(self):
        """
        Analyze dependencies and potential bottlenecks in the workstation sequence
        
        Returns:
            Dict: Mapping of workstations with their dependencies and constraints
        """
        dependencies = {}
        for i, workstation in enumerate(self.sequence):
            dependencies[workstation] = {
                'previous_station': self.sequence[i-1] if i > 0 else None,
                'next_station': self.sequence[i+1] if i < len(self.sequence) - 1 else None,
                'processing_time_estimate': workstation.estimated_processing_time,
                'resource_requirements': workstation.resource_requirements
            }
        return dependencies
    
    def validate_sequence(self):
        """
        Validate the manufacturing sequence for feasibility
        
        Returns:
            bool: Whether the sequence is valid
        """
        # Check for circular dependencies
        # Validate resource availability
        # Ensure all required workstations are present
        pass

class Workstation:
    def __init__(self, station_id, name, capabilities, 
                 estimated_processing_time, resource_requirements):
        """
        Represents a specific manufacturing workstation
        
        Args:
            station_id (str): Unique identifier for the workstation
            name (str): Descriptive name of the workstation
            capabilities (List[str]): Manufacturing processes this station can perform
            estimated_processing_time (float): Average time to complete a process
            resource_requirements (Dict): Resources needed for operation
        """
        self.station_id = station_id
        self.name = name
        self.capabilities = capabilities
        self.estimated_processing_time = estimated_processing_time
        self.resource_requirements = resource_requirements
```

### Example Workstation Sequence
```python
# Example: Manufacturing a complex mechanical component
widget_manufacturing_sequence = WorkstationSequence(
    product_id='WIDGET-001',
    sequence=[
        Workstation(
            station_id='CNC-01', 
            name='CNC Machining', 
            capabilities=['Precision Cutting', 'Drilling'],
            estimated_processing_time=45.5,
            resource_requirements={
                'electricity': 5.2,  # kWh
                'operator_skill_level': 'Advanced'
            }
        ),
        Workstation(
            station_id='WELD-02',
            name='Welding Station', 
            capabilities=['Welding', 'Joining'],
            estimated_processing_time=30.0,
            resource_requirements={
                'electricity': 3.8,  # kWh
                'welding_gas': 0.5,  # m³
                'operator_skill_level': 'Intermediate'
            }
        ),
        # Additional workstations...
    ]
)
```

### Sequence Analysis Methods
- Bottleneck Detection
- Resource Optimization
- Process Time Estimation
- Parallel Processing Identification

### Key Relationships
- `Product` ➡️ `WorkstationSequence`: One-to-One mapping
- `WorkstationSequence` ➡️ `Workstation`: One-to-Many relationship
- `Workstation` ➡️ `Resource`: Many-to-Many relationship

### Performance Metrics
- Throughput Rate
- Station Utilization
- Idle Time
- Process Efficiency

## Bill of Materials (BOM) and Dependency Graph Management

### Comprehensive BOM Modeling System

```python
from typing import List, Dict, Optional
from enum import Enum
from decimal import Decimal
from networkx import DiGraph
import networkx as nx

class MaterialType(Enum):
    RAW_MATERIAL = "Raw Material"
    SEMI_FINISHED = "Semi-Finished"
    FINISHED_PRODUCT = "Finished Product"
    CONSUMABLE = "Consumable"
    PACKAGING = "Packaging"

class BOMItemStatus(Enum):
    ACTIVE = "Active"
    DEPRECATED = "Deprecated"
    PROTOTYPE = "Prototype"
    EXPERIMENTAL = "Experimental"

class BOMItem:
    def __init__(
        self, 
        id: str,
        name: str,
        material_type: MaterialType,
        quantity: Decimal,
        unit_cost: Decimal,
        lead_time: int,  # Days
        status: BOMItemStatus = BOMItemStatus.ACTIVE
    ):
        self.id = id
        self.name = name
        self.material_type = material_type
        self.quantity = quantity
        self.unit_cost = unit_cost
        self.total_cost = quantity * unit_cost
        self.lead_time = lead_time
        self.status = status
        self.alternative_items: List[BOMItem] = []
        self.substitutes: List[BOMItem] = []

class BillOfMaterials:
    def __init__(self, product_id: str, product_name: str):
        self.product_id = product_id
        self.product_name = product_name
        self.dependency_graph = DiGraph()
        self.items: Dict[str, BOMItem] = {}
        self.root_items: List[BOMItem] = []

    def add_item(
        self, 
        parent_id: Optional[str], 
        item: BOMItem
    ):
        self.items[item.id] = item
        
        # Add to dependency graph
        self.dependency_graph.add_node(
            item.id, 
            data=item, 
            name=item.name
        )
        
        if parent_id:
            self.dependency_graph.add_edge(parent_id, item.id)
            
        if not parent_id:
            self.root_items.append(item)

    def get_total_cost(self) -> Decimal:
        return sum(item.total_cost for item in self.items.values())

    def get_max_lead_time(self) -> int:
        return max(item.lead_time for item in self.items.values())

class DependencyGraphAnalyzer:
    @staticmethod
    def analyze_critical_path(bom: BillOfMaterials):
        """
        Analyze critical path for production
        """
        try:
            critical_path = list(nx.dag_longest_path(bom.dependency_graph))
            return {
                'path': critical_path,
                'total_lead_time': sum(
                    bom.items[item_id].lead_time 
                    for item_id in critical_path
                )
            }
        except nx.NetworkXError:
            return None

    @staticmethod
    def find_bottlenecks(bom: BillOfMaterials):
        """
        Identify potential production bottlenecks
        """
        bottlenecks = []
        for node in bom.dependency_graph.nodes():
            in_degree = bom.dependency_graph.in_degree(node)
            out_degree = bom.dependency_graph.out_degree(node)
            
            if in_degree > 1 or out_degree > 1:
                bottlenecks.append({
                    'item_id': node,
                    'item_name': bom.items[node].name,
                    'in_dependencies': in_degree,
                    'out_dependencies': out_degree
                })
        
        return bottlenecks

    @staticmethod
    def check_circular_dependencies(bom: BillOfMaterials):
        """
        Detect circular dependencies in BOM
        """
        try:
            cycles = list(nx.simple_cycles(bom.dependency_graph))
            return cycles if cycles else None
        except nx.NetworkXNoCycle:
            return None

class BOMOptimizer:
    @staticmethod
    def optimize_substitutions(bom: BillOfMaterials):
        """
        Recommend material substitutions
        """
        optimization_suggestions = []
        for item in bom.items.values():
            if item.substitutes:
                best_substitute = min(
                    item.substitutes, 
                    key=lambda x: x.unit_cost
                )
                if best_substitute.unit_cost < item.unit_cost:
                    optimization_suggestions.append({
                        'original_item': item,
                        'recommended_substitute': best_substitute,
                        'cost_saving': item.unit_cost - best_substitute.unit_cost
                    })
        
        return optimization_suggestions
```

## BOM Management Key Features and Design Principles

### 1. Advanced Dependency Tracking
- Directed graph representation of material dependencies
- Multi-level bill of materials support
- Recursive dependency resolution mechanism

### 2. Comprehensive Item Modeling
- Detailed material type classification
- Granular cost and lead time tracking
- Flexible status management
- Support for alternative and substitute items

### 3. Dependency Analysis Capabilities
- Critical path identification for production
- Automated bottleneck detection
- Circular dependency checking
- Optimization suggestion generation

### 4. Intelligent Substitution Management
- Cost-based material substitution recommendations
- Dynamic alternative item tracking
- Automated optimization suggestions

## Integration Strategies

### Production Planning Integration
- Utilize dependency graph for advanced scheduling
- Identify and optimize critical production paths
- Dynamic resource allocation based on dependencies

### Inventory Management Synergy
- Precise material requirement tracking
- Predictive inventory need forecasting
- Lead time and availability management

### Cost Optimization Mechanisms
- Comprehensive product cost calculation
- Intelligent make-or-buy decision support
- Substitution and optimization recommendations

## Advanced Optimization Techniques

### 1. Machine Learning Enhancements
- Predictive material requirement forecasting
- Automated substitution recommendation
- Historical performance-based optimization

### 2. Real-time Dependency Management
- Dynamic BOM updates
- Instant impact analysis of changes
- Automated notification systems

### 3. Visualization and Reporting
- Interactive dependency graph visualization
- Comprehensive reporting tools
- What-if scenario modeling

## Recommended Future Developments
1. Advanced machine learning integration
2. Real-time collaborative BOM editing
3. Blockchain-based BOM verification
4. AI-driven optimization algorithms
5. Enhanced visualization and interaction tools

## Performance and Scalability Considerations
- Efficient graph-based data structure
- Lazy loading of dependency information
- Caching mechanisms for complex calculations
- Horizontal scalability support

## Relationship and Interaction Diagram

```
User -----> Production Order
           |
           v
Equipment <--> Production Step
Worker ---/

Inventory Item --> Product
Supplier -------> Purchase Order
                  |
                  v
                  Inventory Item

Production Order --> Quality Checks
                    |
                    v
                    Defects
```

## Key Design Principles

1. **Loose Coupling**: Models are designed to be independent yet interconnected
2. **Immutability**: Prefer immutable data structures where possible
3. **Rich Domain Model**: Each model contains both data and behavior
4. **Event-Driven**: Models emit events for state changes
5. **Audit Trail**: Every significant change is tracked

## Detailed Model Relationships and Interactions

### 1. User-Centric Relationships
- Users have specific roles with granular permissions
- Different roles have access to different parts of the system
- Permissions control view, edit, and delete capabilities

### 2. Resource Allocation Dynamics
- Equipment and Workers are dynamically assigned to Production Orders
- Skill matching ensures optimal resource utilization
- Performance metrics track individual and collective efficiency

### 3. Inventory and Supply Chain Integration
- Products define their Bill of Materials (BOM)
- Inventory Items track stock levels and quality
- Suppliers provide materials with performance tracking

### 4. Production Planning Complexity
- Production Orders break down into specific Production Steps
- Each step requires specific skills and equipment
- Quality checks are integrated at multiple stages

### 5. Quality Management Workflow
- Defects are tracked from detection to resolution
- Root cause analysis helps prevent future issues
- Corrective actions are documented and monitored

## Advanced Features in Model Design

### 1. Predictive Capabilities
- Models include fields for machine learning predictions
- Historical data supports advanced analytics
- Performance metrics enable predictive maintenance

### 2. Flexible Configuration
- Enum-based status and type fields allow easy extension
- Generic interfaces support multiple manufacturing contexts
- Configurable thresholds for alerts and interventions

### 3. Comprehensive Tracking
- Timestamp fields for every significant event
- Audit log capabilities built into model design
- Support for regulatory compliance and detailed reporting

## Recommended Implementation Approach

1. **Base Model Creation**
   - Implement abstract base classes
   - Define core interfaces
   - Create type hints and validation

2. **ORM Mapping**
   - Use Django's ORM for database interactions
   - Create migrations for each model
   - Implement database constraints

3. **Business Logic Layer**
   - Add methods for complex interactions
   - Implement domain-specific validations
   - Create service classes for cross-model operations

4. **Event and Notification System**
   - Implement signal-based event tracking
   - Create notification mechanisms
   - Support real-time updates

## Next Development Steps
1. Finalize model definitions
2. Create database migrations
3. Implement serialization methods
4. Develop comprehensive test suites
5. Build initial API endpoints

Would you like me to elaborate on any specific aspect of the model design or discuss the implementation strategy?
