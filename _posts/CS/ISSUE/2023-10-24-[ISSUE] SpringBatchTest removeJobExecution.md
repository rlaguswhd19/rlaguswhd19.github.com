---
title : "[ISSUE] SpringBatchTest removeJobExecution()
tags :
 - issue

---

# SpringBatchTest

스프링 배치 테스트를 작성하는 도중에 테스트가 실행되지 않았다. 처음엔 테스트가 오래 걸리는 정도였으나 이후에는 14분째 실행이 되지 않았다. 그래서 작성된 테스트에 디버깅을 했다.

~~~java
@DisplayName("증분 색인 테스트")
@SpringBatchTest
@BatchTestAnnotation
class BbsPartialIndexingJobTest extends TestContainerConfig {
    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;
    @Autowired
    private JobRepositoryTestUtils jobRepositoryTestUtils;
    @Autowired
    @Qualifier("bbsPartialJob")
    private Job bbsPartialJob;

    @BeforeEach
    void setUp() {
        this.jobLauncherTestUtils.setJob(bbsPartialJob);
        this.jobRepositoryTestUtils.removeJobExecutions();
    }

    @Test
    @DisplayName("성공")
    void bbsPartialJob() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addLocalDateTime("date", LocalDateTime.now())
                .addLong("from", 0L)
                .addString("alias", Const.IndexName.Alias.BBS)
                .addString("type", "test")
                .toJobParameters();

        JobExecution jobExecution = jobLauncherTestUtils.launchJob(jobParameters);

        Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
    }
}
~~~

`setUp` 함수를 보면 `jobRepositoryTestUtils.removeJobExecutions()` 함수를 호출하는데 이게 모든 JobRepository를 조회해서 삭제하고 있다.

~~~java
public void removeJobExecutions() {
		List<String> jobNames = this.jobRepository.getJobNames();
		for (String jobName : jobNames) {
			int start = 0;
			int count = 100;
			List<JobInstance> jobInstances = this.jobRepository.findJobInstancesByName(jobName, start, count);
			while (!jobInstances.isEmpty()) {
				for (JobInstance jobInstance : jobInstances) {
					List<JobExecution> jobExecutions = this.jobRepository.findJobExecutions(jobInstance);
					if (jobExecutions != null && !jobExecutions.isEmpty()) {
						removeJobExecutions(jobExecutions);
					}
				}
				start += count;
				jobInstances = this.jobRepository.findJobInstancesByName(jobName, start, count);
			}
		}
	}
~~~

db의 모든 job들을 가져와서 삭제를 하고있었다. 개발 db에 반영되어 meta 데이터를 삭제처리하기 때문에 지금까지 쌓여있던 데이터들이 많아 오래걸렸던것! 그래서 test에서 실행한 job 하나만 삭제를 했다.

<br/>

~~~java
@DisplayName("증분 색인 테스트")
@SpringBatchTest
@BatchTestAnnotation
class BbsPartialIndexingJobTest extends TestContainerConfig {
    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;
    @Autowired
    private JobRepositoryTestUtils jobRepositoryTestUtils;
    @Autowired
    @Qualifier("bbsPartialJob")
    private Job bbsPartialJob;
    private JobExecution jobExecution;

    @BeforeEach
    void setUp() {
        this.jobLauncherTestUtils.setJob(bbsPartialJob);
    }

    @AfterEach
    void deleteMetaData() {
        this.jobRepositoryTestUtils.removeJobExecution(jobExecution);
    }

    @Test
    @DisplayName("성공")
    void bbsPartialJob() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addLocalDateTime("date", LocalDateTime.now())
                .addLong("from", 0L)
                .addString("alias", Const.IndexName.Alias.BBS)
                .addString("type", "test")
                .toJobParameters();

        jobExecution = jobLauncherTestUtils.launchJob(jobParameters);

        Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
    }
}
~~~







