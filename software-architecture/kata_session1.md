### Notes 

`19:00`
```
0. Identify the business level requirements. Through the BRD, spot the domain level informations. (Bring clarity to those statements). 
    (How important is it..)
[Domain Level understanding]


1. First generate set of Architecture requirements. 
So, How would identifiy the architectural requirements, You should be able to identify the non-functional requirements. 
[NFRs]


===
```
Ask for domain level information if you dont know
```
===

```
`Domain (BRD) + Architeture characteristics that system need to support`

```

Define things in within your own domain, also have to take NFR things into consideration.  

Architectural Design  + Domain 
(Security is one of the key constraints)

The architectural characteristics will lead you to the appropriate architectural styles. 

What are architectural styles? Monolithic, Microservices and etc. 

Each arch style supports set of arch characteristics. 

Dont go for too many characteristics.. this will be counterintutive

Match up architectural characteristics to architectural styles. 

DIfference between scalability vs elasticity
scalability: (Ability to handle concurrent users)
Elasticity: (Ablity to burst out when things are under load)


```


===
```

Are there any business domain specific architectural constaints that you need to take care of such as: As example outlined, reputation index. { Find out from domain experts. }. This may not directly impact architetural domain. 

```
===

All the list will inform architect that what kind of architectural style that you need to inform. 


---
So, How do you approach it?. 
```
Component Identification.
-------------------------

Identify who is responsible?
Data and data relationships.

First time, you wont get it right. You will have iterate on it. 


Component decomposition. 
First set of empty bucket. 

```
###
```
Avoid Entity Trap. 

(Dont add manager to entity.)

```

You need to be intested in the workflows than the data and data relationships. 

Work flow driven approach in (DDD).
Event stroming design


Work flow to Entity

Actor / Action Approach. (Applicable across all designs)
(Use case)

System.


<br>

Variety of approach. 

<br>

you are staring with nothing, 

From Actor / Action model approach. ( you need to quetion the session model)
<br>
I guess, this similar to Use case model on what actions are going to be performed from the user prespective.

<br>

---

<br>

Going through this exercise, you are identifying the component interactions. you are not done yet. 


<br>

go back to component design and architecture characteristics. and verify. 

<br>

The point is DDD alone is not sufficient, you would need to actor/action model that combines with architectual characteristics to validate (or to findout ) which component needs what kind of architectural characteristics. 


--- 

Use Stiky notes. 

--
<br>
Key note: Dont get emotionally attached to the diagram for now. Use sticky, have lo-fi prototyping as much as possible. 

<br>

It would be impossible to come up with the perfect design now shortly. 

Look at the suitability. 

<br>

Implement general topology. 

<br>

There are several different ways to attack a particular problem. 

<br>

Component decomposition. 

<br>

Figure out where the state lives.. then iterate again. 

@40:00

By isolating data itself, i would be 

@45:00

```

This is still an component decomposition diagram. 

Come up with an diagram, call out communication structure, whether it is synchronous or asynchronous structure. 

```


c4 
simon brown diagramming technique
```
https://c4model.com/
4+1

```
Kata Idea 
Kata catalog

```
Document s/w architecture.  on why?. (Document architecture decision) Why? (Underlying details)

The diagram may show how you solved , not why?. 

In architecture, why is more important than how?. 


```


