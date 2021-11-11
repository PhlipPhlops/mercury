## Mercury - Prelimnary
Inspired by human-to-human communication, the mercury protocol is a developing idea for peer-to-peer neural network communication.
Loosely, I see no need in layer-wise neural network construction aside from computational efficiency. As a pattern, it is divorced from the inspiration of organic neural networks. Instead, in an organic network, each neuron seems to act independently, fishing out it's own connections; it receives input on the dendrites, which move independetly looking for synaptic output hormones [citation needed lol] and when it's received enough input signal in its dendritic connections it pulses, emitting signals down its axons (output).

In current layer-wise neural network construction, the total number of nuerons are preset by the engineer, and the only thing modifiable are the weights of signal input (along with other engineered concepts like learning rates).

Inspired by Conway's Game of Life, I aim to construct a simple algorithm for neurons to independently find their way to new connections, severing old ones if necessary, such that signal from an input set of nuerons (perception neurons) can find its way to requisite output neurons (ie, motor neurons). Let's begin.

## The neuron, vs the perceptron.
The perceptron model is conveniently (computationally conviently) abstracted away into matrix representation.
[x1]--[w1] \
[x2]--[w2] - (()) --- [y]
[x3]--[w3] /

This perceptron with 3 inputs and one output (every perceptron has one output, though that output can be read from many further neurons)
Can be represented by a matrix dot product:
[X * W] gives you the sum of all the weights multiplied by their associated input. Then you pass it into some activation function to get your perceptrons output
y = F(X * W)

Notice in this case, this matrix operation operates on two 1-dimensional vectors X and W
Take a collection of neurons, and you can apply the same reasoning [my intellectual laziness is striking me here and I am not going to elaborate just yet] to do the same operation on 2-dimensional vectors, so that activations between two layers of neurons can be calculated all at once.

As far as I loosely understand, the same layerwise pattern allows for back-propagation for easier gradient descent; another engineered (and useful!) concept that I choose to ignore for the sake of aesthetic and intellectual curiosity.

### The layerless network
I was able to prove in a previous experiment the function of a non-layerwise neural networks using a matrix to represent connections between neurons in a graph. A 10x10 matrix, as in, a matrix with 10 total neurons set up like such:
The values stored on this matrix are the weight of the connection or edge. The neuron on the left is a [From] neuron, on the top is a [Two] neuron. I passed in the input values of a XOR function into neurons i1 and i2, allowed the signal no propogate along the edges, modified by weight [I can verify this once I find my old code on my Fedora instance, I believe], and I read the output from the neuron o1. I calculated the error across a batch of inputs, then perturbed all weight randomly by a small amount, generating 10 new networks. Then I ran the inputs again, selecting the least error network, killing the rest, and continuing to pertub and repopulate the networks. The result? An evolutionary layer-agnostic neural network that was able to compute the output of a XOR function (impossible for a single perceptron to do).

   i1 i2 a b c d e f g o1 
   _  _  _ _ _ _ _ _ _ _
i1|
i2|
 a|
 b|
 c|
 d|
 e|
 f|
 g|
o1|

This is only inspiration. I have different machinations.

## The neuron structure and representation
What we're building is, essentially, a graph for the purposes of allowing signal to propogate down its edges.
There are two efficient graph representations, useful for different purposes.
#### The adjacency matrix
Useful for finite numbers of nodes (ie, in a layerwise representation), queries like "does this specifici edge exist between nodes a and b?" can be looked up in O(1) time, but space-wasteful for sparse graphs. Allows for matrix operations (which can be GPU optimized)
#### The adjacency list
Every node stores its own list of adjacent nodes. Memory efficient for sparse graphs, flexible number of nodes (a neuron can be added at any time and added to another neurons adjacency list), specific queries are not efficient, require Breadth-first or Depth-first search algorithm to hunt down a node; but that's ok. We don't need to run queries, we only need to allow signals to propagate. Additionally, we need the flexibility to add new neurons at anypoint (Neurogenesis.)

### Our Neuron, an a-neuron
As such I'm proposing a neuron structure, with attempt to make this as computationally efficient as possible (leveraging go pointers)
Each neuron is a structure akin to a node on a linked list.
struct {
    axonNodes: map[address of next neuron; weight value (learned paramters)]
    threshold: float, positive value # modifiable value (learned parameter) used for the "do i fire?" computation
    actionPotential: float # current sum of all signals received on this iteration.
    receiveSignal(int): method, a previous neuron calls this method and passes in its activation * the weight between that neuron and this one.
        This method adds to the actionPotential sum, then makes a check; if actionPotential > threshold, sendSignal()
    sendSignal(): method, iterate through every axonNode and call its receiveSignal method with its associated weight passed in.
    relax(): method, reset actionPotential to 0 (used to refresh the whole network)
    connect(*neuron-address): method, called by outside neurons, adds themselves to this neurons axonNodes list for future fires.
}


### Cerebrospinal Fluid methods (CSF - aka, methods of the playing field that the neurons live in)
Within the brain is a playingfield outside the domain of the neuron. For example, one neuron fires, and it leaves a trail of neurotransmitters leaking from the axon; whatever transmitters don't get reabsorbed by synaptic connecitons; this leaves a trail for further neurons to send their dendrites to sniff out. Other methods will be stored in this domain as well, like the passage of time and relaxation. The signal propogation will be done here, as well as the evolution and signal relaxation of pathways.

#### Fire.
Starting with a singal starting neuron (or perhaps, on a clock cycle between a handful of neurons, the "Will cortex", my own unproven idea), run a breadfirst search to propogate signals down the network. Why breadth-first? It allows the network to fire roughly in layers, from "inside to outside", ordering is determined on the a quasi-evolutionary history, from the "lizard brain" of older neurons to the "outer shell" of newer ones as the network evolves.

#### Pathing structure
An [unproven] intuition, inspire by the Lightning Network Gossip-layer and the organic neurons behavior of sending out dendrites (not axons) to 'sniff' out neuro-transmitters to find a neuron to connect to 
> Neurons look for connections to read from, not connections to write to.
For this reason, when an a-neuron fires, it should add its memory address to a pool of "just fired" addresses that new neurons can read from in order to form new connections.

For this case, we can have a single dynamically sized list. Any time a neuron's ATP passes it's threshold and it fires the sendSignal() method, it adds its own memory address to this dynamic list. Then as new neurons are touched, they reference this list to decide which neurons they want to add themselves to. Perhaps they'll randomly generate a binary mask the length of the list (at its current state), and connect to all nodes that pass through the filter. The connection will leverage a neuron method connect()

#### Relaxation
Pulses in the brain regressed as proton-pumps within a neuron rebalance the electrical activity, moving the neuron away from the signal threshold. In the same breadth-first motion of the fire() signal, go through and reset the action potential of each neuron in the brain (or, push it down a bit, not fully on each clock cycle.)


