# Grounded language paper reading

[pdf document](./grounded langauge learning fast and slow.pdf)

[TOC]

##  Abstract

Recent work has shown that large text-based neural language models acquire a
surprising propensity for one-shot learning. Here, we show that an agent situated
in a simulated 3D world, and endowed with a novel dual-coding external mem-
ory, can exhibit similar one-shot word learning when trained with conventional
RL algorithms. After a single introduction to a novel object via visual perception
and language (“This is a dax”), the agent can manipulate the object as instructed
(“Put the dax on the bed”), combining short-term, within-episode knowledge of
the nonsense word with long-term lexical and motor knowledge. We find that, un-
der certain training conditions and with a particular memory writing mechanism,
the agent’s one-shot word-object binding generalizes to novel exemplars within
the same ShapeNet category, and is effective in settings with unfamiliar numbers
of objects. We further show how dual-coding memory can be exploited as a signal
for intrinsic motivation, stimulating the agent to seek names for objects that may
be useful later. Together, the results demonstrate that deep neural networks can ex-
ploit meta-learning, episodic memory and an explicitly multi-modal environment
to account for fast-mapping, a fundamental pillar of human cognitive development
and a potentially transformative capacity for artificial agents.

### Abstract comment 

1. large text based NL models acquire a surprising propensity for one-shot learning (requires few training data to train a model)
2. a novel dual coding external memory
   1. exhibit similar one shot word learning when trained with conventional RL algorithm
3. under certain training conditions and with a particular memory writing mechanism,
   1. the agent's one shot word object binding generalizes to novel exemplars(范本) 



## Introduction section

### general notes

1. language models exhibit one or few shot learning 
   1. they can adapt their knowledge to new information 
2. fast-mapping
   1. the ability to bind new word to an unfamiliar object after a single exposure
   2. studied facet of child language learning (Susan Carey and Elsa Bartlett. Acquiring a single new word. Papers and Reports on Child Language
      Development, 1978.)
3. goal of this paper
   1. enable an embodied learning system to perform fast-mapping
      1. agent can learn names of entirely unfamiliar objects in a single exposure. and then immediately apply this knowledge to carry out instructions based on those objects. 
4. how the agent observe the world and react
   1. active perception of raw pixels 
   2. learn to respond to linguistic stimuli by executing sequences of motor actions. 
   3. trained by a combination of conventional RL and predictive (semi-supervised) learning

### learning to fast map novel names to novel objects

1. author finds out that learning to fast map novel names to novel objects in a single episode relies on semi-supervised prediction mechanism and a novel form of external memory
2. inspired by dual coding theory of knowledge representation
   1. the agent can exhibit both slow word learning and fast mapping 

## ShapeNet

Angel X Chang, Thomas Funkhouser, Leonidas Guibas, Pat Hanrahan, Qixing Huang, Zimo Li,
Silvio Savarese, Manolis Savva, Shuran Song, Hao Su, et al. Shapenet: An information-rich 3D
model repository. arXiv preprint arXiv:1512.03012, 2015.



## factors affects generalisation ability of agent 

1. the number of unique objects observed by the agent during training and the temporal aspect of its perceptual experience of those objects contribute critically to its ability to generalize
2. particularly the ability to execute fast mapping with entirely novel objects 
3. a dual-coding memory schema can provide a more effective basis to derive a signal for
   intrinsic motivation than a more conventional (unimodal) memory



## experiments issue

1. why does the instruction phase only allow one instruction but not a combination of instruction? (would the number of instructions during the training process affects the result?)