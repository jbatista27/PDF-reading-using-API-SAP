FUNCTION zfm_read_manual_pdf.
*"----------------------------------------------------------------------
*"*"Interface local:
*"  IMPORTING
*"     VALUE(IV_FILE_DIRECTORY) TYPE  STRING
*"----------------------------------------------------------------------
  "Objetos
  DATA: lo_http_client TYPE REF TO if_http_client.

  "Tabelas
  DATA: lt_binary TYPE STANDARD TABLE OF x255.

  "Estruturas
  DATA: lv_str     TYPE string,
        lv_qtn_str TYPE i,
        lv_qtn     TYPE i.

  "Variáveis
  DATA: lv_response    TYPE string,
        lv_xstring     TYPE xstring,
        lv_binary      TYPE xstring,
        lv_file_length TYPE i,
        lv_vbeln       TYPE vbrk-vbeln.

  CALL METHOD cl_http_client=>create_by_url
    EXPORTING
      url                = 'https://sandbox.api.sap.com/mlfs/api/v2/image/ocr'
    IMPORTING
      client             = lo_http_client
    EXCEPTIONS
      argument_not_found = 1
      plugin_not_active  = 2
      internal_error     = 3
      OTHERS             = 4.

  IF sy-subrc <> 0.
    ev_erro = abap_true.
    RETURN.
  ENDIF.

  "setting request method
  lo_http_client->request->set_method('POST').
  "adding headers
  lo_http_client->request->set_header_field( name = 'content-type' value = 'multipart/form-data; boundary=---011000010111000001101001' ).
  "API Key for API Sandbox
  lo_http_client->request->set_header_field( name = 'APIKey' value = '' ).

  CALL FUNCTION 'GUI_UPLOAD'
    EXPORTING
      filename   = iv_file_directory
      filetype   = 'BIN'
    IMPORTING
      filelength = lv_file_length
    TABLES
      data_tab   = lt_binary.

  CALL FUNCTION 'SCMS_BINARY_TO_XSTRING'
    EXPORTING
      input_length = lv_file_length
    IMPORTING
      buffer       = lv_xstring
    TABLES
      binary_tab   = lt_binary
    EXCEPTIONS
      OTHERS       = 1.

  IF sy-subrc <> 0.
    ev_erro = abap_true.
    RETURN.
  ENDIF.

  lo_http_client->request->append_cdata(
    `-----011000010111000001101001`
    && cl_abap_char_utilities=>cr_lf
    && `Content-Disposition: form-data; name="files"; filename="file.pdf";`
    && cl_abap_char_utilities=>cr_lf
    && cl_abap_char_utilities=>cr_lf ).
  lo_http_client->request->append_data( lv_xstring ).
  lo_http_client->request->append_cdata(
    cl_abap_char_utilities=>cr_lf
    && `-----011000010111000001101001--` ).

  CALL METHOD lo_http_client->send
    EXCEPTIONS
      http_communication_failure = 1
      http_invalid_state         = 2
      http_processing_failed     = 3
      http_invalid_timeout       = 4
      OTHERS                     = 5.

  IF sy-subrc = 0.

    CALL FUNCTION 'SAPGUI_PROGRESS_INDICATOR'
      EXPORTING
        percentage = 50
        text       = 'Lendo PDF'.

    CALL METHOD lo_http_client->receive
      EXCEPTIONS
        http_communication_failure = 1
        http_invalid_state         = 2
        http_processing_failed     = 3
        OTHERS                     = 5.

  ENDIF.

  IF sy-subrc <> 0.
    ev_erro = abap_true.
    RETURN.
  ENDIF.

  lv_response = lo_http_client->response->get_cdata( ).

  ENDIF.
  
ENDFUNCTION.
