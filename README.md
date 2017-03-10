# NDlib - Network Diffusion Library

NDlib provides implementations of several spreading and opinion dynamics models.
It is implemented in Python 2.7 (support for Python 3.x pending).

At the moment NDlib makes available the following models

**EPIDEMICS**

1. **SI** *(SIModel)*
 - W. O. Kermack and Ag McKendrick. A Contribution to the Mathematical Theory of Epidemics. Proceedings of the Royal Society of London. Series A, Containing Papers of a Mathematical and Physical Character, 1927.
2. **SIR** *(SIRModel)*
 - W. O. Kermack and Ag McKendrick. A Contribution to the Mathematical Theory of Epidemics. Proceedings of the Royal Society of London. Series A, Containing Papers of a Mathematical and Physical Character, 1927.
3. **SIS** *(SISModel)*
 - W. O. Kermack and Ag McKendrick. A Contribution to the Mathematical Theory of Epidemics. Proceedings of the Royal Society of London. Series A, Containing Papers of a Mathematical and Physical Character, 1927.
4. **Threshold** *(ThresholdModel)*
 - M. Granovetter. Threshold models of collective behavior. American Journal of Sociology, 1978  
5. **Kertesz Threshold** *(KerteszThresholdModel)*
 - Karsai M., Iniguez G., Kaski K., and Kertesz J., Complex contagion process in spreading of online innovation. Journal of the Royal Society, 11(101), 2014
6. **Independent Cascades** *(IndependentCascadeModel)*
 - D. Kempe, J. Kleinberg, and E. Tardos. Maximizing the Spread of Influence through a Social Network. In KDD, 2003.
7. **Profile** *(ProfileModel)*
8. **Profile Threshold** *(ProfileThresholdModel)*
 
**OPINION DYNAMICS**

9. **Voter** *(VoterModel)*
 - Peter Clifford and Aidan Sudbury. A model for spatial conflict. Biometrika, 60(3), 1973. 
10. **QVoter** *(QVoterModel)*
 - Claudio Castellano, Miguel A Mu~noz, and Romualdo Pastor-Satorras. Nonlinear q-voter model. Physical Review E, 80(4), 2009.  
11. **Majority Rule** *(MajorityRuleModel)*
 - S Galam. Real space renormalization group and totalitarian paradox of majority rule voting. Physica A, 285, 2000.  
12. **Snajzd** *(SznajdModel)*
 - Katarzyna Sznajd-Weron and Jozef Sznajd. Opinion evolution in closed community. International Journal of Modern Physics C, 11(06), 2000. 
13. **Cognitive Opinion Dynamics** *(CognitiveOpDynModel)*
 - Francesca Giardini, Daniele Vilone, and Rosaria Conte. Consensus emerging from the bottom-up: the role of cognitive variables in opinion dynamics. Frontiers in Physics, 2015  

## Installation

In order to install the library just download (or clone) the current project and copy the ndlib folder in the root of your application.
Alternatively use pip:
```bash
sudo pip install ndlib
```

## Example usage

Import the selected diffusion model with
```python
import ndlib.models.VoterModel as m
```

Generate/load a graph with the [networkx](https://networkx.github.io/) library
```python
import networkx as nx
g = nx.erdos_renyi_graph(1000, 0.1)
```
Initialize the model on the graph
```python
model = m.VoterModel(g)
```
Configure the nodel initial status
```python
import import ndlib.models.ModelConfig as mc
config = mc.Configuration()
config.add_model_parameter('percentage_infected', 0.2)
model.set_initial_status(config)
```
Request a single iteration of the simulation
```python
it_id, it_status = model.iteration()
```
or a bunch of iterations
```python
it_bunch = model.iteration_bunch(bunch_size=10)
``` 

Each model can assing multiple statuses to nodes. 
Is it possible to retrive the map used by a given model to identify the available status with
```python
model.get_status_map()
```

## Rationale behind the implemented models

- All models inherit from ```ndlib.models.DiffusionModel```

- Model configuration are defined by a ```ndlib.models.ModelConfig``` object that handle:
	- **model** parameter through, ```add_model_parameter(name, value)```
	- **node** configuration through ```add_node_configuration(param_name, node_id, param_value)```
	- **edge** configuration through ```add_edge_configuration(param_name, edge, param_value)```

- NDlib describes diffusion models as agent-based simulations occurring at discrete time: once configured the desired model and selected the target network, subsequent iterations will provide to the user the current status of each node.

- At each iteration are returned only the nodes (and current status) that changed their previous configuration. 

### Model configuration
Every model needs few parameters to be executed, in particular:

| Model  | Parameters | Description |
| ------------- | ------------- | ------------- |
| **Sznajd** | - | - |
| **Voter**  | - | - |
| **Q-Voter** | model:q | Number of neighbours that affect the opinion of an agent |
| **Majority** | - | - |
| **Cognitive Opinion Dynamics** | model:I | External information value |
| | model:T_range_min | Minimum of the range of initial values for node parameter T |
| | model:T_range_max | Maximum of the range of initial values for node parameter T |
| | model:B_range_min | Minimum of the range of initial values for node parameter B |
| | model:B_range_max | Maximum of the range of initial values for node parameter B |
| | model:R_fraction_negative |Fraction of individuals having the node parameter R=-1 |
| | model:R_fraction_neutral | Fraction of individuals having the node parameter R=0 |
| | model:R_fraction_positive | Fraction of individuals having the node parameter R=1 |
| **Independent Cascades** | edges:threshold | Edge threshold (optional)|
| **Threshold** | nodes:threshold | Node threshold (optional)  |
| **Profile**   | nodes:profile | Node profile (optional)  |
| **Profile-Threshold** | nodes:threshold | Node threshold (optional) |
| | nodes:profile** | Node profile (optional) |
| **Kertesz Threshold** | nodes:threshold | Node threshold (optional)  |
| | model:adopter_rate| Exogenous adoption rate |
| | model:blocked | Percentage of blocked nodes | 
| **SI** |  model:beta  | Infection rate |
| **SIS** | model:beta  | Infection rate |
| | model:lambda | Recovery rate |
| **SIR** | model:beta  | Infection rate |
| | model:gamma | Recovery rate |

All parameters are specified within each method description and retrievable through
```python
model.get_model_parameters()
```

## Visualize simulation Results

NDlib comes with basic visualization facilities embedded in ```ndlib.viz.DiffusionTrend```.

```python
import networkx as nx
from bokeh.io import show
import ndlib.models.ModelConfig as mc
import ndlib.models.epidemics.SIRModel as sir
from ndlib.viz.DiffusionTrend import VisualizeDiffusion

g = nx.erdos_renyi_graph(1000, 0.1)
model = sir.SIRModel(g)
config = mc.Configuration()
config.add_model_parameter('beta', 0.001)
config.add_model_parameter('gamma', 0.01)
config.add_model_parameter("percentage_infected", 0.05)
model.set_initial_status(config)
iterations = model.iteration_bunch(200)
viz = VisualizeDiffusion(model, iterations)
p = viz.plot()
show(p)
```

## Implement new models
Implement additional models is simple since it only requires to define a class that:
- implement the partial abstract ```class ndlib.models.DiffusionModel```
- redefine the ```__init__()``` method to provide model details
- implement the ```iteration()``` method specifying its agent-based rules 

### Structure Example
```python
from ndlib.models.DiffusionModel import DiffusionModel

class MyModel(DiffusionModel):

    def __init__(self, graph):
    	super(self.__class__, self).__init__(graph)
        self.available_statuses = {
            "Susceptible": 0, 
            "Infected": 1
        }
		self.parameters = {"model:param1": "descr", "node:param2": "descr", "edge:param3": "descr"}
        self.name = "MyModel"
    
    def iteration(self):
    
    	self.clean_initial_status(self.available_statuses.values())

    	# if first iteration return the initial node status
        if self.actual_iteration == 0:
            self.actual_iteration += 1
        return 0, self.status
    
        actual_status = {node: nstatus for node, nstatus in self.status.iteritems()}
        for u in self.graph.nodes():
            # evluate possible status changes using the model parameters (accessible via self.params)
            # e.g. self.params['beta'], self.param['nodes']['threshold'][u], self.params['edges'][(id_node0, idnode1)]
        
        # identify the changes w.r.t. previous iteration
        delta = self.status_delta(actual_status)
        # update the actual status and iterative step
        self.status = actual_status
        self.actual_iteration += 1
        
        # return the actual configuration (only nodes with status updates)
        return self.actual_iteration - 1, delta
```
If you like to include your model in NDlib (as well as in [NDlib-REST](https://github.com/GiulioRossetti/ndlib-rest)) feel free to fork the project, open an issue and contact us.
