# ElasticSearch_Study

엘라스틱 서치 공부용 프로젝트

## 주요 구성요소

    엘라스틱 서치 ( Java )
    로그 스태시   ( JRuby )
    비츠        ( Go )
    키바나       ( JavaScript )
    크로스플랫폼을 지원하는 언어를 기반으로 작성돼있다.

## Rest

    웹상의 모든 리소스에 URI를 부여하고 활용하는 아키텍쳐 
    모든 리소스를 일관된 규칙으로 접근할 수 있어 애플리케이션을 모듈이나 기능별로 분리하기 쉬워진다.
    해당 형태를 고수하는 API 형태로 사용되는 것이 RestAPI
    메소드 종류로는 POST, GET, PUT, DELETE 등등이 있다

|     |     |
|-----|-----|
|     |     |

|     |     |
|-----|-----|
| id | type: :integer |
| indexed_at | type: :date |
| interview_process | Contents::InterviewsRepository.multi_field_index_options |
| review_status_id | type: :integer |
| rejection_reason_id | type: :integer |
| rejection_message | type: :text |
| company_pick | type: :boolean |
| interview_referral_id | type: :integer |
| duration_days | type: :integer |
| offer_status_id | type: :integer |
| interview_date | type: :date |
| created_at | type: :date |
| updated_at | type: :date |
| pending_at | type: :date |
| approved_at | type: :date |
| draft_at | type: :date |
| rejected_at | type: :date |
| city_id | type: :integer |
| city_level2_id | type: :integer |
| questions |do |
| id | type: :integer |
| question | Contents::InterviewsRepository.multi_field_index_options |
| answer | Contents::InterviewsRepository.multi_field_index_options |
| ratings.interview_difficulty | type: :integer |
| ratings.overall_integer | type: :integer |
| employee.job_type_id | type: :integer |
| employee.job_rank_id | type: :integer |
| employee.years_employed | type: :integer |
| employee.years_of_experience | type: :integer |
| employee.occupation_id | type: :integer |
| employee.occupation_level2_id | type: :integer |
| company.id | type: :integer |
| company.name | type: :text |
| company.nicknames | type: :text |
| company.logo | type: :text |
| company.industry_id | type: :integer |
| company.industry_level2_id | type: :integer |
| company.headquarter_city_id | type: :integer |

end