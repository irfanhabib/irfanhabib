---
layout: post
title:  "Our Journey with Apache Airflow at GoCardless"
tags: [data-engineering, airflow]
---

At GC, we are deeply committed to efficient, reliable, and scalable data processing and have been using Airflow for several years. Initial users of Airflow were just Business Intelligence team and Data Science, today, there are thirteen teams that use the Airflow instance managed by our team for mission critical workflows. As we have expanded we are facing some challenges that are calling for a more robust solution.

## The Hurdles We're Facing

1. **Lack of Zero-downtime deployment:** This might a consequence of how Airflow is deployed in our infrastructure. Airflow and all DAG code that run in Airflow live in a single Github repository. We containerise this and deploy it to a GKE based cluster. Deployments are automated, and update two environments sequentially `staging` and `production`. They occur every time something is merged to `master` via a [Tekton](https://tekton.dev/) pipeline. As the number of teams that use Airflow has scaled, so have merges to `master`. Therefore resulting in more deployments. Every deployment will tear down all workers deployed and replace them with the new image. However, if the workers were running any tasks, those tasks would be killed. This is not a problem if tasks are short lived, however, at GC, as the scale and complexity of our data warehouse has grown so have the [dbt](https://www.getdbt.com/) based transformation that BI runs on Airflow periodically have grown in complexity as well. These can take up to an hour to run. It was mainly the BI team that as impacted by repeated deployments. My team addressed this by adding a SIGTERM trap to the Airflow workers which causes them remove themselves from the scheduling pool and give a chance to complete the task at hand. We also increased the worker's [terminationGracePeriod](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-terminating-with-grace) in the kubernetes deployment to five hours.
    
1. **Lack of Observability:** When a task fails, our goal is for an actionable alert to be generated, providing the relevant team with enough context to investigate and resolve the issue. Presently, our system lacks the finesse to deliver such functionality reliably. We use Sentry for alerting and have added alerting rules based on a label that identifies the DAG and route them to the team that owns the DAG. However, if an exception occurrs at a lower level, such as the Airflow Scheduler these labels are missing in Sentry and the alert ends up with Data Infra team which adds to their toil.
    
1. **Task Dependencies:** Airflow is effectively a single python virtual environment, therefore all DAG code and their dependencies need to be mutually compatible. This is a major issue for us as teams are blocked from upgrading their code to the latest libraries as other DAGs might be incompatible. Sometimes teams want to leverage a new library version that is incompatible with the version required by Airflow itself! As can be observed in this snippet ![Airflow dependencies](/assets/img/airflow.png)
    To address this issue, we need better DAG isolation or split up our single Airflow instance into multiple isolated team specific instances.
    
1. **Lack of Elastic Scaling:** This might be a consequence on how we deployed Airflow at GC which is very inefficient. We deploy a static set of workers in GKE. The current setup does not support elastic scaling, and all workers are a single node pool. Therefore, if a single DAG is memory intensive, we need to scale all Airflow workers to the min. memory required by that single DAG. Moreover, The node pool is static and only utilised after midnight. Therefore during the day most workers are idle.
    
1. **Development UX:** Developers are finding it difficult to develop a task locally and have it run in production as expected. Certain DAGs require access to Google Cloud Platform resources which individual user's might not have access to personally.
    
1. **Zero-downtime upgrades:**  Upgrading Airflow is always challenging even for minor patch releases like 2.3 -> 2.4. Often the database changes required for an upgrade are not backwards compatible and if things go wrong it's hard to revert back.
    

## Looking Forward

My goal is not to discredit Apache Airflow. It has been instrumental in shaping our data operations. However, as we continue to grow and mature, it's clear we need to start exploring next-generation workflow orchestration solutions that align more closely with our future-oriented vision and operational demands.

I'm excited about the journey ahead, about finding a tool that can rise to these challenges, and about the improvements we can deliver to our teams and stakeholders. As we continue to innovate, we'll share our insights, experiences, and, hopefully, our successes in future posts. Stay tuned for updates!
