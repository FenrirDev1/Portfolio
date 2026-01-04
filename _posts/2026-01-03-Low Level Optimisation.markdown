---
layout: post
title: Low Level Optimisation
permalink: /LLO
yamlName: LLO
show: true
---

### Low Level Optimisation

This project has the folowing features: 


```
Memory Management:
- Memory Tracker
    - Walking the Heap
    - Canaries
- Memory Pools
    - Fixed Memory Pool
    - Custom Arena Pool

Spatial Partioning
- Regions
- Threading
    - Thread Pooling
```

Here are some more in depth:

<details markdown="1">
<summary markdown="1">
### Arena Pool
</summary>

### Why?
The reason for a Arena pool came from regions and regionlists.

Intially once I implemented spatial partioning with regions I was allocating memory to each region equal to the total number of objects as theoretically one region could contain all objects.

One solution was estimating and then growing the region list if it does happen to contain to much however I didn't like that approach as it felt too fuzzy to me.

This meant for each region past the first one I was wasting memory equal to the amount of regions * number of cubes * pointer size (8bytes).

The way this projected worked made perfect sense for a arena pool.

> **1 Object could only be destroyed not created**  
> **2 The Object amount was defined via a define and therfore was constant**

### Implementation

The arena allows all the object pointers for each list to sit side by side, filling up the arena each frame when remaking region lists for spatial partioning.

So all I needed was a start point and a size to loop through so multithreading still worked.

Therefore I created a custom "list" struct for the arena.

<details markdown="1">
<summary>
Region List code Snippit
</summary>

```c++
struct RegionList
{
private:
	ColliderObject** firstElement = nullptr; //First Elements Pointer
public:
	long double size = 0; //How many Elements in this Region

	//Resets list back to empty variables
	void resetList() {
		firstElement = nullptr;
		size = 0;
	}
	ColliderObject* getElement(int index) {
		if (index > size) { throw "Error Index out of Bounds"; }
		ColliderObject* boxElement = firstElement[index];
		return boxElement;
	}
    //Allocates Pointer to Arena, increments size and allocates first element(if first element)
	void pushBack(ColliderObject* element)
	{
		//If this is the first element -> Keep size at 0 and save the element to Arena
		if ((firstElement == nullptr) && (size == 0))
		{
			void* buf = Arena::AllocateArenaRegion(sizeof(ColliderObject*));
			ColliderObject** element2 = new(buf) ColliderObject*;
			*element2 = element;
			firstElement = element2;
			size++;
		}
		else {
			void* buf = Arena::AllocateArenaRegion(sizeof(ColliderObject*));
			ColliderObject** element2 = new(buf) ColliderObject*;
			*element2 = element;
			size++;
		}
	}
};
```
</details>

The Arena implemention is rather simple as its just a bump pointer and filling up the memory with allocations. 
<details markdown="1">
<summary>
Arena Code Snippit
</summary>

```c++
//Region Array setup

//Using Arena for regions
#define REGIONARENASIZE (NUMBER_OF_OBJECTS*sizeof(void*))

std::array<char, REGIONARENASIZE> Arena::_arenaRegion;
size_t Arena::_sizeRemainingRegion = REGIONARENASIZE;
int Arena::_bumpPointerRegion = 0;

//Allocation
void* Arena::AllocateArenaRegion(size_t size)
{

	if (_sizeRemainingRegion >= size)
	{
		char* pMem;
		//decrement sizeremaining by size
		_sizeRemainingRegion -= size;
		//Get the arena's location
		pMem = &_arenaRegion.at(_bumpPointerRegion);
		//Bump the pointer
		_bumpPointerRegion += size;

		return pMem;
	}
	else {
		throw std::bad_alloc();
        }
}
//Reseting
void Arena::EmptyArenaRegion()
{
    //Doesn't Empty memory since reusing it is the same.
	_bumpPointerRegion = 0;
	_sizeRemainingRegion = REGIONARENASIZE;
}

```

</details>

### Results:

This lightweight regionlist struct allowed for super fast looping for the threads each frame alongside a super cheap reset/wipe. Due to not reallocating memory every frame and not wasting memory with too many allocations. This allowed for a perfectly efficent memory solution as the arena fills up fully each frame as all the objects find their regions and then reseting.

In this project I made my memory pools static classes, however in different project I would look into allowing them be created for easier use.
Also the data used in this Arena (pointers) is a simple type so my arena doesn't implement calling a deconstructor, this would be changed if it was needed.

</details>