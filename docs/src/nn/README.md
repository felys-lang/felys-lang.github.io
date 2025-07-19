# Neural Network

The embedded neural network training functionality consists of two core components: automatic differentiation and dynamic computation graph construction. The former is essentially an application of the multivariate chain rule. The latter is more interesting, as it can consistently construct a connected graph even when control flows make the computation non-functional. This section will cover both components, supported by relevant calculus.
