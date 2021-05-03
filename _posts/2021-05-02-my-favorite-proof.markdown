---
layout: post
title:  "My favorite proof method"
date:   2021-05-02 17:49:12 -0400
categories: jekyll update
description: Graph theory has some unique proofs
---
<style TYPE="text/css">
code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}
</style>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [['$','$'], ['\\(','\\)']],
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'] // removed 'code' entry
    }
});
MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS_HTML-full"></script>

Graph theory is very unique area in math, especially if you're coming from a more calculus/linear algebra background. Half the time, it seems like you are just drawing stuff on a piece of paper without really having any idea what you're doing. When I took my first graph theory course, I had the privilge of working with a brilliant professor, [Jim Geelen][jim-geelen]. It was in this class where I learnt a new proof method known as **Minimal Counterexample**.

You can think of minimal counterexample as a mix of contradiction and induction. Suppose you are trying to prove some statement $p$. With minimal counterexample, you will assume that $p$ is false. As such, there must be at least on counterexample that makes the statement false. Pick the "smallest" counterexample $C$. Smallest in this context can be quite vague, but it is one of the most important parts of the proof. In graph theory, "smallest" usually means a graph with as few edges (or vertices) as possible. Likewise for numbers, it could mean the smallest possible number. Since $C$ is minimal, it means that for any mathematical object "smaller" then $C$ (fewer edges, or smaller number), the statement will be true. From these 2 remarks, we can deduce a contradiction to prove the original statement. The proof method sounds really complicated at first. If it's difficult to wrap your head around this, we'll go through two examples.

Statement: Every natural number $n \ge 2$ can be written as a product of primes.

_Proof:_ Suppose otherwise and let $m$ be a minimal counter example. Then it means that:
1. $m$ is a natural number that cannot be written as a product of prime and,
2. Any natural number $n$, with $n < m$ **can** be written as a product of primes

From our first remark, $m$ must isn't a prime number. So, we can find other natural numbers $p, q$ such that we have $m = p \times q$. Note that we must have $a < m$ and $b < m$. Therefore by our second remark, we get that both $p, q$ are written as a product of primes. But since have $m = p \times q$, it must be that $m$ can be written as a product of primes also, a contradiction. _QED_

Hopefully this example gives you a sense what "smaller" means in this context. The next example gets a little more involved, and uses graph theory as a motivation. For the proof below, I will use a know fact that for a graph $G = (V,E)$, if $e$ is an edge in $G$ that isn't in a cycle, then $G - e$ is not connected.

**Fact**: If $G = (V,E)$ is a non-empty graph with $\|E\| \ge \|V\|$, then $G$ has a cycle.

_Proof_: Suppose otherwise and let $G = (V,E)$ be a counterexample with as few _edges_ as possible. This means that:
1. $G$ is non-empty and $\|E\| \ge \|V\|$ but has no cycle.
2. If $H$ is a non-empty graph with $\|E(H)\| \ge \|V(H)\|$ that has _fewer_ edges than $G$, then $H$ must have a cycle.

Since $G$ is non-empty and $\|E\| \ge \|V\|$, $G$ must have an edge. Let $e$ be such edge. By our fact mentioned above, we have that $G - e$ is not connected.

Let $H_1, \dots H_k$ be the components of $G - e$. Since $G$ doesn't have a cycle, then surely none of $H_1, \dots H_k$ have cycles either. Then it means that for each $H_i$, we have that $\|E(H_i)\| \le \|V(H_i)\|$. Observe the following:

$$
\begin{align}
|V(G)| &\le |E(G)| \\
&= 1 + |E(H_1)| + \dots + |E(H_k)| \\
&\le 1 + (|V(H_1) - 1) + \dots + (|V(H_k)| - 1) \\
&= 1 - k + |V(H_1)| + \dots + |V(H_k)| \\
&= 1 - k + |V(G)|
\end{align}
$$


However, this would imply that $k \le 1$, which is a contradiction. _QED_

The proof above was definetly more involved, but it can show the power of proof by minimal counterexample. 

[jim-geelen]: http://www.math.uwaterloo.ca/~jfgeelen/