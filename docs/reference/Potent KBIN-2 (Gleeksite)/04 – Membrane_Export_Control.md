SET payload_files_list=PAYLOAD_FILES_9PLUS

DETECT export_trigger WHEN user_request in {"export payload","export domain files","export KBIN","show files","show GleekSite files","print KBIN","print payload files","print GleekSite files"}
DETECT runtime_export_request WHEN user_requests_files_in_range(0-8) OR user_requests_runtime_files_explicitly=true
DETECT runtime_exclusion_query WHEN user_asks_reason_for_runtime_exclusion

IF export_trigger=true EMIT mode_set(export_domain_only) TO Runtime_Core
IF export_trigger=true SET export_requested=true

IF session.export_mode=true FILTER file_list TO payload_files_list
IF session.export_mode=true OUTPUT filenames_and_literal_content_only
IF session.export_mode=true BLOCK commentary_during_export
IF session.export_mode=true BLOCK interpretation_during_export
IF session.export_mode=true ENFORCE domain_only_output

IF runtime_export_request=true REFUSE runtime_file_export
IF runtime_export_request=true MAINTAIN export_mode_state

IF runtime_exclusion_query=true OUTPUT single_line_neutral_explanation
