---
title: "MNIST Neural Nets"
classes: wide
use_math: true
author_profile: false
tags:
  - prog
---

During my sophomore year of highschool, I had a job working for an astrophysicist at Los Alamos National Laboratory. He has a completely free program for local teenagers interested in learning python, which I had participated in several years prior. In my job, I wrote new chapters in the online textbook he used to teach it (he also makes this publically availably). I wrote six chapters in the book, as well as overhauling another one. Unfortunately, most of the chapters were deemed to lengthy or complex to make the final cut of the book, and now languish in an "incomplete chapter" section. This post will be about the most complex of these, and the programs I wrote to illustrate the topic: neural networks.

I did a deep dive into both the math behind the neural networks (which I didn't end up including in the chapters due to its complexity) and the programming skills necessary to implement it. I used the NetworkX library to represent the neural nets, which doesn't have any built-in machine learning methods. This was all programmed by hand, and explained in a way hopefully parsable to a novice programmer. 

I started with a very simple case: predicting polynomial data. As this was meant to be a gentle introduction to neural networks, I opted to use a genetic algorithm instead of backpropagation (which I'll explain later). This meant starting with all of the parameters of the network in a random configuration, and then tweaking them randomly over iterations until they produce an acceptably close fit to the data. This was a relatively simple endeavor, and the program and final result are shown below:

<details>
<summary>
These are the helper functions, which are imported at the top of many of the programs shown below.
</summary>
{% highlight python linenos %}

import networkx as nx
import numpy as np
import matplotlib.pyplot as plt
import random # for weights

def generate_blank_net(layer_sizes):
    """creates a network with a specified number of neurons in each layer and random weights."""
    blank_net = nx.DiGraph()
    for i in range(len(layer_sizes)):
        for j in range(layer_sizes[i]):
            neuron = f'l{i}_{j}' # properly formats the neuron names
            blank_net.add_node(neuron)
        blank_net.add_node(f'l{i}_b', activation=1)
    blank_net.remove_node(f'l{len(layer_sizes) - 1}_b') # remove bias neuron in output layer

    for n_1 in blank_net.nodes:
        n_1_layer = int(n_1.split('_')[0][1:])
        for n_2 in blank_net.nodes:
            # we need to check if the neurons are in adjacent layers, and make sure that the second neuron isn't a bias.
            n_2_layer = int(n_2.split('_')[0][1:])
            if n_1_layer + 1 == n_2_layer and n_2[-1] != 'b':
                blank_net.add_edge(n_1, n_2, weight=random.random()*2 - 1)
    
    return blank_net

def generate_layer_dict(net):
    """make a dictionary of a given neural net keyed by layer."""
    layer_dict = {}
    for neuron in net.nodes:
        neuron_layer = int(neuron.split('_')[0][1:]) 
        if neuron_layer not in layer_dict.keys():
            layer_dict[neuron_layer] = []
        layer_dict[neuron_layer].append(neuron)
    return layer_dict

def calc_next_layer(previous_layer_index, net, layer_dict, output=False):
    """given a neural net and the index of the last calculated layer, iterate the calculation one layer further."""
    for n_1 in layer_dict[previous_layer_index + 1]: # n_1 is the neuron that gets a new activation
        activation = 0
        if n_1[-1] != 'b':
            for n_2 in layer_dict[previous_layer_index]:
                activation += net.nodes[n_2]['activation'] * net[n_2][n_1]['weight']
            if output:
                net.nodes[n_1]['activation'] = activation
            else:
                net.nodes[n_1]['activation'] = np.tanh(activation)
        else:
            net.nodes[n_1]['activation'] = 1

    return net

def calc_all_layers(input_layer, net, layer_dict): 
    """returns the output layer given a neural net and an input layer."""
    if len(input_layer) != len(layer_dict[0]) - 1: # make sure the input layer is valid
        raise Exception(f'Input layer must be the same length as the first layer of nodes! (input is {input_layer} and first layer is {len(layer_dict[0]) - 1} long)')
    input_layer.append(1) # add bias neuron value
    for i in range(len(input_layer)):
        net.nodes[layer_dict[0][i]]['activation'] = input_layer[i]
    input_layer.pop(-1)

    for i in range(len(layer_dict) - 1):
        output = (i == len(layer_dict) - 2)
        net = calc_next_layer(i, net, layer_dict, output=output)

    output_layer_neurons = layer_dict[np.max(list(layer_dict.keys()))]
    output_layer_neurons.sort(key=lambda x: x.split('_')[1])
    output_layer = []
    for neuron in output_layer_neurons:
        output_layer.append(float(net.nodes[neuron]['activation']))
    
    return output_layer

{% endhighlight %}
</details>

<details>
<summary>
This is the main genetic algorithm program.
</summary>
{% highlight python linenos %}

from helper_functions import *

def main():
    poly_coeffs = [0.4, -1, 1, 1]
    domain = [-2,1.5]
    num_points = 100
    noise = 0.6
    num_children_per_net = 50
    num_survive = 5
    variance = 0.1
    generations = 150
    net_dimensions = [1,5,5,5,1]
    
    first_parent = generate_blank_net(net_dimensions)
    xs, ys = generate_polynomial_data(poly_coeffs, domain, num_points, noise)
    next_gen = make_next_gen([first_parent], xs, ys, num_children_per_net, num_survive, variance) # create first generation
    
    inputs = []
    expected_outputs = []
    for i in range(len(xs)):
        inputs.append(xs[i][0])
        expected_outputs.append(ys[i][0])  # reformat xs and ys for matplotlib
    
    for i in range(generations):
        next_gen = make_next_gen(next_gen, xs, ys, num_children_per_net, num_survive, variance)
        print(f'The RSS is {eval_total_cost(next_gen[0], xs, ys) * 1000} after generation {i+1}.') # the times 1000 makes the RSSs a reasonable value
        
        outputs = []
        for i in range(len(xs)):
            outputs.append(calc_all_layers(xs[i], next_gen[0], generate_layer_dict(next_gen[0])))
        plt.clf()
        plt.plot(inputs, expected_outputs)
        plt.plot(inputs, outputs)
        plt.savefig('fitted-curve.png')
        print('Figure saved to fitted-curve.png')          

def eval_RSS(net_outputs, expected_outputs):
    """gets RSS for a given set of outputs"""
    RSS = 0
    for i in range(len(net_outputs)):
        for j in range(len(net_outputs[i])):
            RSS += ((net_outputs[i][j] - expected_outputs[i][j]) ** 2) / len(net_outputs[i]) # make error independant of output layer size
    RSS /= len(net_outputs) # make error independant of data set size
    return RSS

def eval_total_cost(net, input_layers, expected_output_layers):
    """properply formats inputs to eval_RSS"""
    net_outputs = []
    layer_dict = generate_layer_dict(net)
    for i in range(len(input_layers)):
        net_outputs.append(calc_all_layers(input_layers[i], net, layer_dict))
    cost = eval_RSS(net_outputs, expected_output_layers)
    return cost / len(input_layers)
                       
def make_children(net, variance, num_children):
    """creates a specificied number of tweaked copies of a given net"""
    children = []
    for i in range(num_children):
        child = nx.DiGraph(net)
        for edge in child.edges:
            child.edges[edge]['weight'] += random.random() * variance - variance / 2
        children.append(child)
    return children

def make_next_gen(parents, input_layers, expected_output_layers, num_children_per_net=20, num_survive=1, variance=0.1):
    """creates many of copies of a net, tweaks them slightly, then picks out the best ones to create the next generation."""
    children = parents.copy() # make sure RSS doesn't go up
    best_children = []
    costs = []
    for parent in parents:
        loop_children = make_children(parent, variance, num_children_per_net)
        for child in loop_children:
            children.append(child)
    for child in children:
        costs.append(eval_total_cost(child, input_layers, expected_output_layers))
        
    for i in range(num_survive):
        best_child_index = costs.index(np.min(costs))
        best_children.append(children[best_child_index])
        children.pop(best_child_index)
        costs.pop(best_child_index)

    return best_children
                                 
def generate_polynomial_data(coefficients, domain, num_points, noise=0):
    """returns two formatted lists: one for inputs, the other for expected outputs"""
    xs = np.linspace(domain[0], domain[1], num_points)
    ys = []
    xs_lists = [] # because of numpy syntax, we have to make two xs arrays
    for i in range(len(xs)):
        # add variation to the data to give fitting a thourough test
        xs[i] += random.random() * (domain[1] - domain[0]) * noise / (0.1 * num_points) - (domain[1] - domain[0]) * noise / (0.2 * num_points)
        xs_lists.append([xs[i]])
        # input_layers is a list of lists, so this formats the inputs properly
        y = 0
        for j in range(len(coefficients)):
            y += (coefficients[j] * (xs[i] ** j)) + (random.random() * noise) - (noise / 2)
        ys.append([y]) # same as above but for outputs
    return xs_lists, ys

if __name__ = '__main__':
  main()

{% endhighlight %}
</details>

<img src='/assets/images/fitted-curve.svg' style="width:150%">
