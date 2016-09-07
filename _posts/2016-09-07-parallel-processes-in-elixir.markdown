---
layout: post
title:  "Parallel processes in Elixir"
date:   2016-06-17 13:20:37 +0200
categories: elixir development
---
When you start to learn Elixir, you can read everywhere that processes are light to create and very fast to execute.

So I wanted to demonstrate this statement with a very simple comparison: *Enum.map/2* and *pmap/2* where _pmap/2_ is the [Dave Thomas’s implementation](https://pragprog.com/book/elixir13/programming-elixir-1-3) of _parallel map_.

Let’s begin to create a project shell:
{% highlight bash %}
$ mix new map_benchmark
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/map_benchmark.ex
* creating test
* creating test/test_helper.exs
* creating test/map_benchmark_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd map_benchmark
    mix test

Run "mix help" for more commands.
{% endhighlight %}

Go to _map_benchmark_ folder and open the file _mix.exs_ with your preferred editor; add the benchmark library we are going to use to measure times:

{% highlight elixir %}
defp deps do
  [{:benchee, "0.2.0"}]
end
{% endhighlight %}

Let’s update dependencies:

{% highlight bash %}
$ mix do deps.get, deps.compile
Running dependency resolution
* Getting benchee (Hex package)
  Checking package (https://repo.hex.pm/tarballs/benchee-0.2.0.tar)
  Using locally cached package
==> benchee
Compiled lib/benchee/config.ex
Compiled lib/benchee.ex
Compiled lib/benchee/repeat_n.ex
Compiled lib/benchee/formatters/console.ex
Compiled lib/benchee/benchmark.ex
Compiled lib/benchee/time.ex
Compiled lib/benchee/statistics.ex
Generated benchee app
{% endhighlight %}

Now open the file _lib/map_benchmark.ex_ and add functions to benchmark:

{% highlight elixir %}
defmodule MapBenchmark do

  defp pmap(collection, func) do
    me = self
    collection
    |> Enum.map(fn e -> spawn(fn -> (send me, {self, func.(e)}) end) end)
    |> Enum.map(fn pid -> receive do {^pid, result} -> result end end)
  end

  defp work(e) when is_number(e) do
    e * e
  end

  def run do
    collection = 1..100

    Benchee.run(%{time: 10},
      [{"map", fn -> Enum.map(collection, &(work(&1))) end},
       {"pmap", fn -> pmap(collection, &(work(&1))) end}
      ])
  end
end
{% endhighlight %}

Enter in _IEx_ playground:

{% highlight bash %}
$ iex -S mix
Erlang/OTP 18 [erts-7.3]  [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Compiled lib/map_benchmark.ex
Generated map_benchmark app
Consolidated List.Chars
Consolidated Collectable
Consolidated String.Chars
Consolidated Enumerable
Consolidated IEx.Info
Consolidated Inspect
Interactive Elixir (1.2.5) - press Ctrl+C to exit (type h() ENTER for help)
{% endhighlight %}

and run our first benchmark:

{% highlight bash %}
iex(1)> MapBenchmark.run
Benchmarking map...
Benchmarking pmap...

Name                                    ips        average    deviation         median
map                                51817.27        19.30μs   (±131.45%)        17.00μs
pmap                                1911.44       523.17μs    (±34.87%)       510.00μs

Comparison:
map                                51817.27
pmap                                1911.44 - 27.11x slower
:ok
{% endhighlight %}

_pmap/2_ is slower than map…
We can try to simulate a slightly harder work; modify the work function as follow:

{% highlight elixir %}
defp work(e) when is_number(e) do
  :timer.sleep(1)
  e * e
end
{% endhighlight %}

Reload module…

{% highlight bash %}
iex(2)> r MapBenchmark
lib/map_benchmark.ex:1: warning: redefining module MapBenchmark
{:reloaded, MapBenchmark, [MapBenchmark]}
{% endhighlight %}

and rerun:

{% highlight bash %}
iex(3)> MapBenchmark.run
Benchmarking map...
Benchmarking pmap...

Name                                    ips        average    deviation         median
pmap                                 380.73      2626.53μs    (±39.62%)      2492.00μs
map                                    4.95    202148.06μs     (±3.04%)    199992.00μs

Comparison:
pmap                                 380.73
map                                    4.95 - 76.96x slower
{% endhighlight %}

Just with a little heavy work load, _pmap/2_ is clearly faster than _Enum.map/2_. Let’s try with more work:

{% highlight elixir %}
defp work(e) when is_number(e) do
  :timer.sleep(10)
  e * e
end
{% endhighlight %}

Update and run again:

{% highlight bash %}
iex(4)> r MapBenchmark
lib/map_benchmark.ex:1: warning: redefining module MapBenchmark
{:reloaded, MapBenchmark, [MapBenchmark]}
iex(5)> MapBenchmark.run
Benchmarking map...
Benchmarking pmap...

Name                                    ips        average    deviation         median
pmap                                  78.40     12754.56μs     (±7.44%)     12692.00μs
map                                    0.83   1210244.00μs     (±1.47%)   1209205.50μs

Comparison:
pmap                                  78.40
map                                    0.83 - 94.89x slower
{% endhighlight %}

So, 95% faster than _Enum.map/2_.

### RESULTS

Use processes parallelization when work load is not shamefully trivial. The more heavy is the job to do, the more performance you can get.
