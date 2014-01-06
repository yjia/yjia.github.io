---
layout: post
title: "Separate Feature Release From System Deployment"
description: ""
category: continuous_delivery
tags: [continuous deplivery, deployment, release, feature toggle]
---


## Problem

Release is a painful activity in some software projects. Business needs some features on a particular date. When they asked the development team for them, development team often prepares a tested system package containing the features and deploys that package to the production environment on the due date. 

This approach has several challenges:

*  Preparing the right package with the right feature set. 

   Software often needs to support multiple parallel changes from different business needs. These different needs have different due release date. These due dates may change because of the change of business or external dependencies. It's a challenge for the development team to prepare a right package with the right feature set on a particular date. It's not uncommon that business have to delay the release to wait for the right package from the development team. Development team becomes the "bottleneck" of the process.  

* Large risky release 

	Business often wants a feature to be released as a unit. A feature may include multiple funcitonalities changes. From technical point of view, it may includes large number of code changes or even changes across multiple systems, some of which may be not within the control of the project team. In the approach above, on the due date, all the team of impacted systems have to be carefully coordinated so that the deployment to each impacted systems happen in the right order. 
	
	It will be worse if after the deployment, one crutial feature does not work as expected. It's often hard to diagnose the cause of the problem because the large number of changes introdueced in the new package. If not able to find the cause within permitted time, rolling back to the previous package not only means rolling back all the impacted systems but also other new features in the new package will be rollback as well. All people have to work under high pressure (often late in the night) in the process. 
	
It often causes the distrust between business and development team and causes the software release into a negative cycle: Business may tend to group multiple features into a bigger release and decrease release freqency, which makes the release more painful.

## Causes

### Feature release and the business need
Feature release is making a particular feature usable on a particular date. That's what the business cares about. Business rarely cares whether there's any changes on the system side as long as there's no perceivable changes from the user's point of view.  

### System deployment and the development team need
System deployment is to use the new software packages (including their configuations) to replace the old ones to serve requests. With the help of automation, development team wants to do more frequent deployment because frequent deployment means smaller number of changes per deployment. It'll be helpful to identify/fix issues early and quickly.  

### Coupling between featue release and system deployment

The problem of the approach at the beginging is that it treats system deployment as the way to do feature release. Such a coupling mandates the size and timing of the deployment have to follow the requirementment of the feature release, which will cause the conflict between the business need and development team need.

## Solution

The solution is to decouple the relationship between featue release and system deployment. Let the development team decide the right time to do deployment and let business people decide when to release a feature.

### Feature toggle
Feature toggle is a flag to indicate whether a particular feature/funcitons will be visible/usable to end users. 

When using feature toggle to control the visibility of the new feature, even the code are deployed to production, as long as the toggle is off, the new feature won't be usable, i.e. not released. Therefore, development team could be free to schedule the deployment. They could schedule the deployment anytime before the due date and could schedule multple deployments to control the deployment change size.

When the busines want to use(or trial) the featuer on a particular date, they just need to turn on the toggle. 

The toggle could be implemented in several ways including following:

* Runtime feature toggle
  
  User could change the value of the toggle during system runtime without need of repackage or redeployment.
  
* Deployment time featuer toggle
  
  Toggle value is changable during deployment process. But this change is normally just a simple configuration change rather than a deployment with differnt packages.
  

### Example 

We work on a system that uses customed login page to authenticate user againest users stored in local database. As the number of users increases, in order to meet the compliance requirement of the company, we are required to migrate the authentication mechanism into a ADFS based federated authentication solution. It needs changes in 2 sides:

1. A 3rd party team is responsible for changes in the ADFS side to setup the right configurations for the federation
2. We are responsible for changes in the system to not using the custom login page and to use the Fed-Auth module.

We use the branch by abstraction to implement the Fed-Auth and Custom-Auth behind the same interface and use runtime feature toggle to decide which one will take effect.

It gives us a lot of benefit in scheduling the release. It's because after we deployed the package with the feature to production(with toggle turned off) , we (the development team) handed over the control to the business to decide when to release. Although the feature release date changed multiple times because of business reasons and 3rd party technical reasons, it doesn't cause any trouble for the development team and business feel more confident to handle the changes because they have the control. 


{% include JB/setup %}
