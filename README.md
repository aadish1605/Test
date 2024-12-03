@Test
public void testProcessDataWithNullExclusionType() {
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(null);
    
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, never()).getEfiaExclusion(
        any(EfiaInput.class), 
        any(ExclusionType.class), 
        any(CommonRedisResponse.class), 
        anyString()
    );
    assertTrue(producerProcessingDto.getExclusionList().isEmpty());
}

@Test
public void testProcessDataWithExclusionType() {
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    ExclusionType exclusionType = new ExclusionType();
    Exclusion mockExclusion = new Exclusion();
    
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(exclusionType);
    
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(exclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(mockExclusion);
    
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(exclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(mockExclusion));
}

@Test
public void testProcessDataWithExceptionHandling() {
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    RuntimeException mockException = new RuntimeException("Test Exception");
    
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(mockException);
    
    ExclusionType defaultExclusionType = ExclusionType.builder().build();
    Exclusion errorExclusion = new Exclusion();
    
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(errorExclusion);
    
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(errorExclusion));
    
    verify(log).info(contains("MT545 Exception::"), eq(mockException));
}

@Test
public void testProcessDataWithMultipleExceptions() {
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    RuntimeException generateException = new RuntimeException("Generate Exception");
    RuntimeException exclusionException = new RuntimeException("Exclusion Exception");
    
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(generateException);
    
    ExclusionType defaultExclusionType = ExclusionType.builder().build();
    
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(exclusionException);
    
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    
    verify(log).info(contains("MT545 Exception::"), eq(generateException));
    verify(log).info(contains("MT545 Exception::"), eq(exclusionException));
}

@Test
public void testProcessDataWithNullInputs() {
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    assertThatNoException().isThrownBy(() -> 
        mt545Service.processData(null, null, producerProcessingDto)
    );
    
    assertNotNull(producerProcessingDto.getExclusionList());
}

@Test
public void testProcessDataWithCustomExclusionType() {
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    ExclusionType customExclusionType = ExclusionType.builder()
        .exclusionTypeId(123L)
        .exclusions("CUSTOM_EXCLUSION")
        .build();
    
    Exclusion customExclusion = new Exclusion();
    
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(customExclusionType);
    
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(customExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(customExclusion);
    
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(customExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(customExclusion));
}
