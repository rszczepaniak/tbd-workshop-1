IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   6

   [***link to forked repo***](https://github.com/rszczepaniak/tbd-workshop-1)
   
2. Follow all steps in README.md.

3. From avaialble Github Actions select and run destroy on main branch.
   
4. Create new git branch and:
![img.png](successful-release.png)


5. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    Moduł composer tworzy zarządzane środowisko Apache Airflow w usłudze Google Cloud Composer, które pełni rolę centralnego orkiestratora pipeline’ów danych. Jest on zależny od sieci VPC (depends_on = module.vpc module.vpc) i tworzy dedykowaną podsieć dla klastra Composera (subnet_address = local.composer_subnet_address). Moduł konfiguruje środowisko Airflow w zadanym projekcie i regionie, przypisuje je do sieci VPC (network = module.vpc.network.network_name) oraz ustawia zmienne środowiskowe umożliwiające integrację z Dataproc, GCS i klastrem GKE wykorzystywanym do zadań dbt i Spark.

    Ze względu na zbyt małą quotę musieliśmy usunąć moduł composer.

    Pełen graph przed usunięciem:
    ![img.png](full-graph.png)

    Graph po usunięciu:
    ![img.png](partial-graph.png)

6. Reach YARN UI
   `gcloud compute ssh tbd-cluster-m --project tbd-2025z-3187321 --zone europe-west1-c -- -L 8088:localhost:8088`
   
    ![img.png](yarn-ui-browser.png)

7. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. Description of the components of service accounts
    2. List of buckets for disposal
    
    ***place your diagram here***

8. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 
    
    Nasze szacunki:
    ![img.png](infracost_estimate.png)

    Komentarz na PR:
    ![img.png](infracost_comment.png)

9. Create a BigQuery dataset and an external table using SQL
   ```
create schema if not exists dataset;

create or replace external table dataset.shakespeare
  options (
    format = 'ORC',
    uris = ['gs://tbd-2025z-3187321-data/data/shakespeare/*.orc']
  );
```

    Format ORC nie wymaga oddzielnego `table schema` ponieważ zawiera on informacje o swoim schemacie (to jest nazwy kolumn oraz typy danych) wewnątrz pliku. Dzięki temu Big Data może automatycznie tworzyć strukturę danych bez ręcznego definiowania schematu.

10. Find and correct the error in spark-job.py

    Problem polegał na tym, że nazwa Bucketa była ustawiona na poprzedni projekt. Został zmieniony na
    `DATA_BUCKET = "gs://tbd-2025z-3187321-data/data/shakespeare/"`

    Reszta powinna być w porządku - o tym czy zadziała dowiemy się z logów Dataproc > Jobs > Jobs kiedy już możliwe będzie zreleasowanie projektu.

11. Add support for preemptible/spot instances in a Dataproc cluster

    [link to modified file](https://github.com/rszczepaniak/tbd-workshop-1/blob/master/modules/dataproc/main.tf)

    What was added (at the bottom)
```
preemptible_worker_config {
      num_instances  = 2
      preemptibility = "SPOT"

      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
      }
    }
```    
    
12. Triggered Terraform Destroy on Schedule or After PR Merge. Goal: make sure we never forget to clean up resources and burn money.

Add a new GitHub Actions workflow that:
  1. runs terraform destroy -auto-approve
  2. triggers automatically:
   
   a) on a fixed schedule (e.g. every day at 20:00 UTC)
   
   b) when a PR is merged to main containing [CLEANUP] tag in title

Steps:
  1. Create file .github/workflows/auto-destroy.yml
  2. Configure it to authenticate and destroy Terraform resources
  3. Test the trigger (schedule or cleanup-tagged PR)
     
***paste workflow YAML here***

***paste screenshot/log snippet confirming the auto-destroy ran***

***write one sentence why scheduling cleanup helps in this workshop***
