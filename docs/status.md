---
layout: default
title:  Status
---
## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/ND62gIA778U" frameborder="0" allowfullscreen></iframe>

## Project Summary
The goal of our project is to train an agent to fight in a 1v1 scenario. We would like our bots to evolve to learn to kill the opposing agent.  We are using a Neuroevolution algorithm, which is a form of machine learning that uses genetic algorithms to evolve the structure and edge-weights of neural networks. More specifically, we are using the NeuroEvolution of Augmenting Topologies (NEAT) algorithm which was created by Ken Stanley in 2002 while he was at The University of Texas Austin. We use the distance between both agents and the damage inflicted in our fitness function. We want to decrease the distance between the agents, so it learns to get close to the opponent as well as increase the damage it inflicts onto the opponent.

## Context
As stated before, the NEAT algorithm is a type of genetic algorithm created by Ken Stanley which changes its weight parameters based on the fitness and the diversity among the specimen of each generation by tracking the history.  Although this field of "neuroevolution" (evolving the parameters of a neural net through a genetic, evolutionary algorithm) is old and dates back further than this paper, NEAT introduced the idea of "species" in the population.  
 
The idea of speciation is highly motivated by biological analogues.  Essentially, Stanley, et al. wanted to pursue the biological concept of sexual reproduction and chromosome crossover (where two successful genomes are combined to create a new, hopefully successful genome).  However, this concept proved difficult to implement for neuroevolution, since two equally-successful neural nets may have radically different *structures*, such that chromosome crossover is nonsensical and results in an unsuccessful offspring.  Previous solutions were just to give up on implementing this biologically analogous crossover by only implementing asexual reproduction, or to have the neural net's structure predetermined as a hyperparameter.  But Stanley did not give up.  Ken Stanley is a hardworking, persevering researching, and he (et al.) came up with this idea of speciation - in which genomes in the population are grouped together into species, by a similarity metric.  Only organisms within the same species are selected to mate with each other.

<p><img src="pics/genome.png" alt="" style="width: 400px;"/></p>

## Approach
The NEAT algorithm is a type of genetic algorithm which changes its weight parameters based on the fitness and the diversity among the specimen of each generation by tracking the history via history markers for crossover.  It evolves the species to bring about the best in each species via evolution, and incrementally develops the topology of the structure (adding or removing nodes as possible mutations).  A brief overview of the genetic algorithm: First we create a large population of genomes, N. Each represents a different neural net (genes are the nodes and edges of the neural net).  We then test each genome (by running our Malmo bot with the neural net created from that genome), assign it a fitness score (score of how well the organism did), and select the two members from that species with the best fitness score. Then we crossover the genomes (randomly picking a position in the chromosome then swapping everything after it) and mutate the resulting genomes to form a new generation.

We used a NEAT python [library](https://github.com/CodeReclaimers/neat-python) together with Malmo to teach our agent. To begin we created a 10 x 10 grid made of diamond blocks. We spawn our two agents at the same location and randomize the yaw between 0 and 360. Our world trains the population, evaluates the genomes, and handles the fighter. 
 
## Environment
10 x 10 x 4 diamond blocks enclosure 

![Environment](pics/10X10X10.png)

## Different Approaches
Without armor 

![without_armor](pics/world_no_armor.png)

With Diamond armor and 10 x 5 x 4 diamond block enclosure, giving the opponent regenerative health.

![Diamond_armor](pics/diamond.png)

Our fighter class can make four continuous moves: move, strafe, turn, and attack. It decides these commands based on the neural net’s output.  There are two inputs to the neural net: the agent's distance to the other agent, and the agent's angle to the other agent.

```python
    def run(self):
        while self.agent.peekWorldState().number_of_observations_since_last_state == 0:
            if not self.isRunning():
                return
            time.sleep(0.01)
        agent_state_input = self._get_agent_state_input()
        scaled_state_input = scale_state_inputs(agent_state_input)
        output = self.neural.activate(scaled_state_input)
        if DEBUGGING:
            print("angle {:.2f}; dist {:.2f};   move {:.3f}; strafe {:.3f}; turn {:.3f}; attack {:.3f}".format(*(agent_state_input + output)))
        if self.mission_ended or not self.agent.peekWorldState().is_mission_running:
            return
        self.agent.sendCommand("move {}".format(output[0]))
        self.agent.sendCommand("strafe {}".format(output[1]))
        self.agent.sendCommand("turn {}".format(output[2]))
        self.agent.sendCommand("attack {}".format(0 if output[3] <= 0 else 1))
```

These calculated values are then passed to AgentResult which we use as our fitness function. This assigns a fitness to each genome by giving it a scaled reward for inflicting damage and punishes the agent for elapsed time and the distance between the agent and the enemy.

```python
INFLICTED_DAMAGE_SCALE = 2#40
TIME_SCALE = 0.01#1
DISTANCE_SCALE = 0.01#100

class AgentResult:
	def __init__(self):
		self.last_time = time()
		self.distance_area = 0.0
		self.inflicted_damage = 0
		self.mission_time = 0

	def AppendDistance(self,distance):
		cur_time = time()
		time_dif = cur_time - self.last_time
		self.distance_area += time_dif * distance
		self.last_time = cur_time

	def GetFitness(self):
		# To track progress 
		print "Distance: ", self.distance_area
		print "Inflicted Damage: ", self.inflicted_damage
		return self.inflicted_damage * INFLICTED_DAMAGE_SCALE - (self.mission_time * TIME_SCALE) - DISTANCE_SCALE * self.distance_area

	def SetInflictedDamage(self, damage):
		self.inflicted_damage = damage

	def SetMissionTime(self, time):
		self.mission_time = time
```
 
## NEAT Configuration
 
Our config-fighter file has all the configuration for the NEAT algorithms parameters. We used a population size of 100 with two inputs (relative angle to the enemy and distance) and one hidden input. As of current, we are using relu for our activation function but there are other options available to fine tune the learning process. The neat-python library allows us to specify mutation rate, probabilities of adding or removing an edge or node, aggregation in the neural nets, and much more. 


![NEAT_Config](pics/NEAT_Config_Image.png)

## Evaluation

To evaluate our performance, we run our evolutionary program for 10 hours and see if the final generation has learned to kill the opponent.  So far, we have been unsuccessful at reaching this goal.  In the absence of complete success, we look at our bots' fitness throughout generations and see if there is a positive trend.  So far, we have been lackluster at reaching this goal when our population size was 20.

![fitness_of_ea_org_ovr_time](pics/pop_<100_fitness_each_org.png)

![Running_max_fit_ovr_time](pics/pop_<100_max_fitness_ea_org.png)

![Running_max_fit_ovr_time](pics/pop_<100_max_fitness_over_time.png)

## Improvements 

After increasing the population size to 100 we began to see improvments.

![Progress](pics/18766898_10207011941297448_98452268_o.png)

## Remaining Goals and Challenges
We see that our current prototype is limited in it’s scope seldom does it attack and kill the opponent. We will be trying larger population sizes and longer runs to train our agent. Our current activation function could be improved, instead of using a relu, we could use a leaky relu which would allow us to obtain negative values.
 
In the final project, we hope to completely implemented an arena in which we will be training two agent simultaneously and it will learn to kill the opponent instead of simply locating a static position opponent and running at it. To expand, we hope for the agent to learn how to aim (with a sword and a bow), dodge, and navigate through the environment.
 
In future iterations of this project, we will be trying to increase the complexity of the environment such as adding TNT, lava, or blocks and items that forces the agent to avoid or utilize to increase its chance of winning the battle.

Being an evolutionary algorithm, we are up against the whims of nature and RNG to chance us into better organisms.  Unfortunately, this means we must sit through hours of runs of plateaus, where each generation does not seem to be doing much better than the past.  Tweaking the rates of mutation should help us ameliorate this problem, and we will attack that in the following week.


