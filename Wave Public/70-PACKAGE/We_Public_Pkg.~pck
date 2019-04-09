CREATE OR REPLACE PACKAGE We_Public_Pkg IS

  PROCEDURE Insert_Request_Id(p_Request_Id  NUMBER
                             ,p_Request_Bth NUMBER);
  PROCEDURE Process_Wait_Result(p_Bth_Seq NUMBER);
  FUNCTION Strsub(p_Str   VARCHAR2
                 ,p_Fgstr VARCHAR2
                 ,p_n     NUMBER) RETURN VARCHAR2;
  FUNCTION Strlocation(p_Str      VARCHAR2
                      ,p_Fgstr    VARCHAR2
                      ,p_Fgstrnum NUMBER) RETURN NUMBER;

  FUNCTION Strcount(p_Str   VARCHAR2
                   ,p_Fgstr VARCHAR2) RETURN NUMBER;

END We_Public_Pkg;
/
CREATE OR REPLACE PACKAGE BODY We_Public_Pkg IS
  -- =============================================================================
  -- Copyright(c) : Wave Corporation . All rights reserved.
  -- Application  :  Application
  -- -----------------------------------------------------------------------------
  -- Program name           Creation Date     Original Version   Created by
  -- WE_Public_Pkg            2016-Jul-25                                ZhangTongtao
  -- -----------------------------------------------------------------------------
  -- Usage:
  -- called from Concurrent programs.
  -- -----------------------------------------------------------------------------
  -- Description: 
  -- Script for insert table
  --
  --
  --
  -- Parameter    : Written in each function/procedure section.
  -- Return value : Written in each function/procedure section.
  -- -----------------------------------------------------------------------------
  -- Modification History:
  -- Modified Date     Version         Done by          Change Description

  g_User_Id NUMBER := Fnd_Profile.VALUE('USER_ID');
  -- 常量定义
  Gv_Pkg_Name_c CONSTANT VARCHAR2(50) := 'WE_Public_Pkg.'; -- 包名

  -- 变量定义
  Gv_Process_Step VARCHAR2(2000); -- 测试步骤

  /*
  功能：用于处理请的等待
  */
  PROCEDURE Insert_Request_Id(p_Request_Id  NUMBER
                             ,p_Request_Bth NUMBER) IS
  
  BEGIN
  
    INSERT INTO Scux_Conc_Request_Log
      (Bth_Seq
      ,Request_Id
      ,Ou_Id
      ,Creation_Date
      ,Created_By
      ,Last_Update_Date
      ,Last_Updated_By
      ,Last_Update_Login)
    VALUES
      (p_Request_Bth
      ,p_Request_Id
      ,NULL
      ,SYSDATE
      ,g_User_Id
      ,SYSDATE
      ,g_User_Id
      ,g_User_Id
       
       );
  
  END Insert_Request_Id;
  /*
  功能：用于处理请的等待
  */
  PROCEDURE Process_Wait_Result(p_Bth_Seq NUMBER) IS
    Lb_Request_Complete BOOLEAN;
    Lv_Phase            VARCHAR2(20);
    Lv_Status           VARCHAR2(20);
    Lv_Dev_Phase        VARCHAR2(20);
    Lv_Dev_Status       VARCHAR2(20);
    Lv_Message          VARCHAR2(200);
    Ln_Success          NUMBER;
    Lv_Batch_Name       VARCHAR2(100);
    Lv_Po_Tax           VARCHAR2(100);
    Ln_Rlt              NUMBER;
    Ln_Int_Error_Cnt    NUMBER;
  BEGIN
    DELETE Scux_Conc_Request_Log
     WHERE Creation_Date <= SYSDATE - 7;
    COMMIT;
  
    FOR Cu_Req IN (SELECT Request_Id
                     FROM Scux_Conc_Request_Log
                    WHERE Bth_Seq = p_Bth_Seq) LOOP
      Ln_Rlt        := -1;
      Lv_Phase      := NULL;
      Lv_Status     := NULL;
      Lv_Dev_Phase  := NULL;
      Lv_Dev_Status := NULL;
      Lv_Message    := NULL;
      <<fst_Loop>>
      LOOP
        Lb_Request_Complete := Apps.Fnd_Concurrent.Wait_For_Request(Request_Id => Cu_Req.Request_Id
                                                                   ,INTERVAL   => 10
                                                                   ,Max_Wait   => 0
                                                                   ,Phase      => Lv_Phase
                                                                   ,Status     => Lv_Status
                                                                   ,Dev_Phase  => Lv_Dev_Phase
                                                                   ,Dev_Status => Lv_Dev_Status
                                                                   ,Message    => Lv_Message);
      
        Fnd_File.Put_Line(Fnd_File.Log
                         ,To_Char(SYSDATE
                                 ,'YYYY-MM-DD HH24:MI:SS') || ':lV_PHASE' ||
                          Lv_Phase || ' FOR THE REQUEST ID:' ||
                          Cu_Req.Request_Id);
      
        IF Upper(Lv_Phase) IN ('COMPLETED'
                              ,'已完成'
                              ,'C') THEN
          Ln_Rlt := 0;
          IF Upper(Lv_Status) IN ('C'
                                 ,'NORMAL'
                                 ,'正常') THEN
            Fnd_File.Put_Line(Fnd_File.Log
                             ,To_Char(SYSDATE
                                     ,'YYYY-MM-DD HH24:MI:SS') || '请求(' ||
                              Cu_Req.Request_Id || ')正常运行完毕');
          ELSIF Upper(Lv_Status) IN ('E'
                                    ,'错误'
                                    ,'ERROR') THEN
            Fnd_File.Put_Line(Fnd_File.Log
                             ,To_Char(SYSDATE
                                     ,'YYYY-MM-DD HH24:MI:SS') || '请求(' ||
                              Cu_Req.Request_Id ||
                              ')运行报错具体可查看此请求LOG和相关接口表');
          ELSIF Upper(Lv_Status) IN ('G'
                                    ,'警告'
                                    ,'WARNING') THEN
            Fnd_File.Put_Line(Fnd_File.Log
                             ,To_Char(SYSDATE
                                     ,'YYYY-MM-DD HH24:MI:SS') || '请求(' ||
                              Cu_Req.Request_Id ||
                              ')运行有警告具体可查看此请求LOG和相关接口表');
          END IF;
        
        END IF;
        EXIT Fst_Loop WHEN Ln_Rlt = 0;
      END LOOP;
    END LOOP;
  
  END;

  FUNCTION Strsub(p_Str   VARCHAR2
                 ,p_Fgstr VARCHAR2
                 ,p_n     NUMBER)
  -- =============================================================================
    -- Copyright(c) : Wave Corporation . All rights reserved.
    -- Application  :  Application
    -- -----------------------------------------------------------------------------
    -- Program name                 Creation Date     Original Version      Created by
    --  STRSUB                     2016.MAR.08           B1.0            ZhangTongtao
    -- -----------------------------------------------------------------------------
    -- Usage: Get Data,output data.
    --
    -- -----------------------------------------------------------------------------
    --
    -- Parameter    : Written in each procedure section.
    -- Return value : Written in each procedure section.
    -- -----------------------------------------------------------------------------
    -- Modification History:
    -- Modified Date     Version          Done by            Change Description
  
    -- =============================================================================
   RETURN VARCHAR2 IS
  
    v_Ordseq  VARCHAR2(200);
    v_Str1    VARCHAR2(20);
    v_Str2    VARCHAR2(4000);
    v_Revalue VARCHAR2(400);
    v_Len1    NUMBER;
    v_Len2    NUMBER;
    v_n       NUMBER;
    v_Sum     NUMBER := 0;
    v_Wf      NUMBER := 1;
    v_Wt      NUMBER := 0;
    v_Ns      NUMBER := 0;
  BEGIN
  
    v_Str1 := p_Fgstr;
    v_Len1 := Length(v_Str1);
    v_Str2 := p_Str;
    v_Len2 := Length(v_Str2);
    IF (Substr(v_Str2
              ,v_Len2
              ,1)) <> v_Str1 THEN
      v_Str2 := v_Str2 || v_Str1;
      v_Len2 := v_Len2 + 1;
    END IF;
    FOR v_n IN 1 .. v_Len2 LOOP
    
      IF (Substr(v_Str2
                ,v_n
                ,v_Len1) = v_Str1) THEN
      
        v_Wt := v_n - v_Wf;
      
        v_Ns := v_Ns + 1;
        IF (v_Ns = p_n) THEN
          v_Revalue := To_Char(Substr(v_Str2
                                     ,v_Wf
                                     ,v_Wt));
        END IF;
        v_Wf  := v_n + 1;
        v_Sum := v_Sum + 1;
      END IF;
    END LOOP;
    RETURN v_Revalue;
  END Strsub;

  FUNCTION Strcount(p_Str   VARCHAR2
                   ,p_Fgstr VARCHAR2) RETURN NUMBER IS
    -- =============================================================================
    -- Copyright(c) : Wave Corporation . All rights reserved.
    -- Application  :  Application
    -- -----------------------------------------------------------------------------
    -- Program name           Creation Date     Original Version   Created by
    -- STRCOUNT                  28-MAY-2004       1.0                ZhangTongtao
    -- -----------------------------------------------------------------------------
    -- Usage:
    -- called from Concurrent programs.
    -- -----------------------------------------------------------------------------
    -- Description:
    -- Script for insert table
    --
    --
    --
    -- Parameter    : Written in each function/procedure section.
    -- Return value : Written in each function/procedure section.
    -- -----------------------------------------------------------------------------
    -- Modification History:
    -- Modified Date     Version         Done by          Change Description
    -- 2015-Nov-04       B1.0            SIE ZhangTongtao R12 Upgrade
    -- =============================================================================
    v_Str1    VARCHAR2(20);
    v_Str2    VARCHAR2(4000);
    v_Revalue VARCHAR2(400);
    v_Len1    NUMBER;
    v_Len2    NUMBER;
    v_n       NUMBER;
    v_Sum     NUMBER := 0;
    v_Wf      NUMBER := 1;
    v_Wt      NUMBER := 0;
    v_Ns      NUMBER := 0;
  BEGIN
    v_Str1 := p_Fgstr; --?????
    v_Len1 := Length(v_Str1); --???????
    v_Str2 := p_Str; --???????
    v_Len2 := Length(v_Str2); --?????????
    IF (p_Str IS NOT NULL) THEN
      IF (Substr(v_Str2
                ,v_Len2
                ,1)) <> v_Str1 THEN
        v_Str2 := v_Str2 || v_Str1;
        v_Len2 := v_Len2 + 1;
      END IF;
      FOR v_n IN 1 .. v_Len2 LOOP
        --????????????
        IF (Substr(v_Str2
                  ,v_n
                  ,v_Len1) = v_Str1) THEN
          --???????????
          Dbms_Output.Put_Line('V_STR2:' || v_Str2);
          Dbms_Output.Put_Line('UBSTR(V_STR2,V_N,V_LEN1):' ||
                               Substr(v_Str2
                                     ,v_n
                                     ,v_Len1));
          v_Wt := v_n - v_Wf; --???????
          --DBMS_OUTPUT.PUT_LINE('WZ:' || V_N);
          --DBMS_OUTPUT.PUT_LINE(SUBSTR(V_STR2,V_WF,V_WT));--?????????
          v_Ns  := v_Ns + 1;
          v_Wf  := v_n + 1; --?????????
          v_Sum := v_Sum + 1; --?????????
        END IF;
      END LOOP;
    END IF;
    RETURN v_Sum;
  END Strcount;

  FUNCTION Strlocation(p_Str      VARCHAR2
                      ,p_Fgstr    VARCHAR2
                      ,p_Fgstrnum NUMBER) RETURN NUMBER IS
    -- =============================================================================
    -- Copyright(c) : Wave Corporation . All rights reserved.
    -- Application  :  Application
    -- -----------------------------------------------------------------------------
    -- Program name           Creation Date     Original Version   Created by
    -- STRCOUNT                  28-MAY-2004       1.0                ZhangTongtao
    -- -----------------------------------------------------------------------------
    -- Usage:
    -- called from Concurrent programs.
    -- -----------------------------------------------------------------------------
    -- Description:
    -- Script for insert table
    --
    --
    --
    -- Parameter    : Written in each function/procedure section.
    -- Return value : Written in each function/procedure section.
    -- -----------------------------------------------------------------------------
    -- Modification History:
    -- Modified Date     Version         Done by          Change Description
    -- 2015-Nov-04       B1.0            SIE ZhangTongtao R12 Upgrade
    -- =============================================================================
    v_Str1     VARCHAR2(20);
    v_Str2     VARCHAR2(4000);
    v_Revalue  VARCHAR2(400);
    v_Len1     NUMBER;
    v_Len2     NUMBER;
    v_n        NUMBER;
    v_Sum      NUMBER := 0;
    v_Wf       NUMBER := 1;
    v_Wt       NUMBER := 0;
    v_Ns       NUMBER := 0;
    v_Location NUMBER;
  BEGIN
    v_Str1 := p_Fgstr; --?????
    v_Len1 := Length(v_Str1); --???????
    v_Str2 := p_Str; --???????
    v_Len2 := Length(v_Str2); --?????????
    IF (p_Str IS NOT NULL) THEN
      IF (Substr(v_Str2
                ,v_Len2
                ,1)) <> v_Str1 THEN
        v_Str2 := v_Str2 || v_Str1;
        v_Len2 := v_Len2 + 1;
      END IF;
      FOR v_n IN 1 .. v_Len2 LOOP
        --????????????
        IF (Substr(v_Str2
                  ,v_n
                  ,v_Len1) = v_Str1) THEN
          --???????????
          Dbms_Output.Put_Line('V_STR2:' || v_Str2);
          Dbms_Output.Put_Line('UBSTR(V_STR2,V_N,V_LEN1):' ||
                               Substr(v_Str2
                                     ,v_n
                                     ,v_Len1));
          v_Wt := v_n - v_Wf; --???????
          --DBMS_OUTPUT.PUT_LINE('WZ:' || V_N);
          --DBMS_OUTPUT.PUT_LINE(SUBSTR(V_STR2,V_WF,V_WT));--?????????
          v_Ns  := v_Ns + 1;
          v_Wf  := v_n + 1; --?????????
          v_Sum := v_Sum + 1; --?????????
          IF v_Sum = p_Fgstrnum THEN
          
            v_Location := v_n;
          END IF;
        END IF;
      END LOOP;
    END IF;
    RETURN v_Location;
  END Strlocation;

END We_Public_Pkg;
/
