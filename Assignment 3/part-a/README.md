Pagerank using MapReduce Library
=

map_tasks and reduce_tasks is set by developer equal to 4 and 4.

(Input to the script must not include .txt extension)

```
	./run.sh {filename-without-extension}
```

MapReduce C++ Library
=

This is a fork of cdmh/mapreduce, this fork aims to help the portabillity of the original code, for any questions please contact the original developer.

The MapReduce C++ Library implements a single-machine platform for programming using the the Google MapReduce idiom. Users specify a map function that processes a key/value pair to generate a set of intermediate key/value pairs, and a reduce function that merges all intermediate values associated with the same intermediate key. Many real world tasks are expressible in this model, as shown in the Google paper.

    map (k1,v1) --> list(k2,v2)
    reduce (k2,list(v2)) --> list(v2)

To run on MacOS use

```
clang++ -std=c++11 -I ~/lib/mapreduce/include/detail -I ~/lib/mapreduce/include/  prime.cpp -lboost_system -lpthread -lboost_iostreams -lboost_filesystem
```

Synopsis
-

```cpp
namespace mapreduce {

template<typename MapTask,
		 typename ReduceTask,
		 typename Datasource=datasource::directory_iterator<MapTask>,
		 typename Combiner=null_combiner,
		 typename IntermediateStore=intermediates::local_disk<MapTask> >
class job;

} // namespace mapreduce
```
    
The developer is required to write two classes; `MapTask` implements a mapping function to process key/value pairs generate a set of intermediate key/value pairs and `ReduceTask` that implements a reduce function to merges all intermediate values associated with the same intermediate key.
In addition, there are three optional template parameters that can be used to modify the default implementation behavior; `Datasource` that implements a mechanism to feed data to the Map Tasks - on request of the `MapReduce` library, Combiner that can be used to partially consolidate results of the Map Task before they are passed to the Reduce Tasks, and `IntermediateStore` that handles storage, merging and sorting of intermediate results between the Map and Reduce phases.
The `MapTask` class must define four data types; the key/value types for the inputs to the Map Tasks and the intermediate types.

```cpp
class map_task
{
  public:
	typedef std::string   key_type;
	typedef std::ifstream value_type;
	typedef std::string   intermediate_key_type;
	typedef unsigned      intermediate_value_type;

	map_task(job::map_task_runner &runner);
	void operator()(key_type const &key, value_type const &value);
};
```
      
The `ReduceTask` must define the key/value types for the results of the Reduce phase.

```cpp
class reduce_task
{
  public:
	typedef std::string  key_type;
	typedef size_t       value_type;

	reduce_task(job::reduce_task_runner &runner);

	template<typename It>
	void operator()(typename map_task::intermediate_key_type const &key, It it, It ite)
};
```

Extensibility
-
The library is designed to be extensible and configurable through a Policy-based mechanism. Default implementations are provided to enable the library user to run MapReduce simply by implementing the core Map and Reduce tasks, but can be replaced to provide specific features.

| Policy | Application | Supplied Implementation(s) |
| ------ | ---- | --- |
| `Datasource` | `mapreduce::job` template parameter | `datasource::directory_iterator<MapTask>` |
| `Combiner` | `mapreduce::job` template parameter | `null_combiner` |
| `IntermediateStore` | `mapreduce::job` template parameter | `local_disk<MapTask, SortFn, MergeFn>` |
| `SortFn` | `local_disk` template parameter | `external_file_sort` |
| `MergeFn` | `local_disk` template parameter | `external_file_merge` |
| `SchedulePolicy` | `mapreduce::job::run()` template parameter | `cpu_parallel`, `sequential` |

See the [MapReduce C++ Library](http://cdmh.co.uk/papers/software_scalability_mapreduce/library.php) page for more information, and a sample program.
